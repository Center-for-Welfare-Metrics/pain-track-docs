# Architecture and Repositories

This document describes the repository landscape, runtime architecture, data model, and relationship behavior for the Pain Track ecosystem.

## Repository Landscape

| Area | Old system | Current system |
| --- | --- | --- |
| Frontend repository | `OLD-pain-track-client` | `pain-app-client` |
| Frontend GitHub | https://github.com/Center-for-Welfare-Metrics/OLD-pain-track-client | https://github.com/Herikle/pain-app-client |
| Backend repository | `old-pain-app-server` | `pain-app-server` |
| Backend GitHub | https://github.com/Center-for-Welfare-Metrics/old-pain-app-server | https://github.com/Center-for-Welfare-Metrics/pain-app-server |
| Public frontend | https://cp.pain-track.org | https://www.pain-track.org/ |
| Public backend | https://oldpaintrackserver-840926881618.us-central1.run.app | https://newpainappserver-840926881618.us-central1.run.app |
| Status | Legacy system kept mainly for historical access | Active system |

### Repository Roles

The frontend repositories contain the browser application and user interface.

Examples of frontend responsibilities:

- login screens
- navigation and routing
- data-entry forms
- discussion and bookmark views

The backend repositories expose the REST API and own the business rules.

Examples of backend responsibilities:

- authenticating users
- loading and persisting MongoDB records
- creating episodes, tracks, and segments
- returning data for bookmarks, discussions, and exports

### Deployment Ownership Constraint

For both the old and current frontend deployments, Vercel is tied to repositories under Herikle's personal GitHub ownership because the deployment is using the free plan.

Practical impact:

- Herikle is currently the only person who can deploy frontend changes
- backend deployment is less restricted because pushes to `main` can trigger Cloud Run deployment behavior

## Runtime Architecture

### Old system

- Browser frontend deployed at `https://cp.pain-track.org`
- REST backend deployed at `https://oldpaintrackserver-840926881618.us-central1.run.app`
- MongoDB Atlas database
- Additional calculus service at `https://calculus-840926881618.us-central1.run.app`

The calculus service is an external dependency used for episode-related calculations. Its repository is currently unknown.

### Current system

- Browser frontend deployed at `https://www.pain-track.org/`
- REST backend deployed at `https://newpainappserver-840926881618.us-central1.run.app`
- MongoDB Atlas databases for production and development
- AWS SES template repository documented in [aws_ses.md](aws_ses.md)

The current app uses AWS SES for email delivery, but template management lives in a separate repository and is documented independently.

## Data Architecture

Pain Track uses MongoDB with Mongoose. Relationships are not enforced like SQL foreign keys. The backend combines three main patterns:

- stored `ObjectId` references using `ref`
- Mongoose virtuals for reverse lookups and counters
- manual cleanup in use cases and Mongoose implementations

### Relation Mechanics

#### Direct references

Most ownership links are stored directly in documents.

Examples:

- `patient.creator_id -> user`
- `episode.patient_id -> patient`
- `track.episode_id -> episode`
- `segment.track_id -> track`
- `segment_justification.segment_id -> segment`

#### Virtual populate

Reverse lookups and counts are often provided through virtuals instead of embedded data.

Examples:

- patient virtuals for `episodes_count`, `bookmarked`, and `discussions_count`
- episode virtuals for `patient`, `tracks`, `tracks_count`, `bookmarked`, and `discussions_count`
- track virtual for `segments`
- segment virtual for `justifications`
- discussion virtuals for `user` and `replies_count`

#### Embedded subdocuments

Not every structure is a separate collection. Segment documents embed:

- `intensities`
- `quality`
- `interventions[]`
- `symptoms[]`

#### Manual cascade behavior

Deletes are procedural rather than declarative.

- deleting a patient can trigger episode cleanup and patient bookmark cleanup
- deleting an episode can trigger track cleanup
- deleting a track can trigger segment cleanup

The full object graph is not cleaned automatically in one place. Orphaned records are possible unless each dependent collection is handled explicitly.

#### Soft delete in discussions

Discussion deletion is implemented as soft delete through `deletedAt`. Deleted rows remain in the collection and serialization redacts content rather than removing the document.

## Main Relationship Graph

```text
User
 ├─< Patient
 │    ├─< Episode
 │    │    ├─< Track
 │    │    │    ├─< Segment
 │    │    │    │    ├─< SegmentJustification
 │    │    │    │    └─< Discussion
 │    │    │    └─< Discussion
 │    │    ├─< Discussion
 │    │    └─< EpisodesBookmark >─ User
 │    ├─< Discussion
 │    └─< PatientsBookmark >─ User
 ├─< Prompt
 │    └─< AiGenerated
 ├─< RecoveryPassword
 ├─< SetPasswordCode
 └─< UpdateEmailCode

Discussion
 └─< Discussion   (self-reference through parent_id)
```

## Model Summary

### User

Core fields include `email`, `name`, `password`, `provider`, and `super`.

User-linked records include:

- patients
- episodes
- discussions
- prompts
- patient bookmarks
- episode bookmarks
- recovery password records
- set password codes
- update email codes

The `email` field is uniquely indexed and `toJSON` removes `password`.

### Patient

Stored relation:

- `creator_id -> user`

Effective behavior:

- one user can own many patients
- one patient can have many episodes
- one patient can have many discussions
- one patient can be bookmarked by many users

### Episode

Stored relations:

- `patient_id -> patient`
- `creator_id -> user`

Effective behavior:

- one patient can have many episodes
- one episode can have many tracks
- one episode can have many discussions
- one episode can be bookmarked by many users

### Track

Stored relation:

- `episode_id -> episode`

Effective behavior:

- one episode can have many tracks
- one track can have many segments

### Segment

Stored relation:

- `track_id -> track`

Effective behavior:

- one track can have many segments
- one segment can have many justifications
- symptoms and interventions are embedded inside the segment document

### Segment Justification

Stored relation:

- `segment_id -> segment`

Effective behavior:

- one segment can have many justifications

### Discussion

Stored relations:

- `user_id -> user`
- `patient_id -> patient`
- `episode_id -> episode` when discussion scope is narrower than patient
- `track_id -> track` when discussion scope is narrower than episode
- `segment_id -> segment` when discussion scope is narrower than track
- `parent_id -> discussion` for replies

Effective behavior:

- discussions can belong to patient, episode, track, or segment context
- replies create a self-referential thread tree
- top-level discussions require a title

### Bookmark Collections

Patients bookmarks and episodes bookmarks are many-to-many join collections between users and the domain entities they save.

Important caveat:

- composite indexes exist on `(entity_id, user_id)` but are not declared unique, so duplicate bookmark rows are not prevented at schema level

### Prompt and AI Generated

- one user can own many prompts
- one prompt can have many AI-generated outputs
- there is no clear cleanup path for AI-generated rows when a prompt is deleted

### Auth-Related Temporary Records

Three record types exist for account operations:

- recovery passwords
- set password codes
- update email codes

All are linked to users and are meant to store short-lived verification or recovery state.

## Populate Patterns Used In Practice

The application frequently loads relation data at query time rather than storing nested structures eagerly.

Main patterns:

- patient lists populate `episodes_count`, `bookmarked`, and `discussions_count`
- episode lists populate `tracks_count`, `bookmarked`, and `discussions_count`
- track lists populate `segments`
- full episode export populates `tracks -> segments -> justifications` and `patient`
- discussion lists populate `user` and `replies_count`
- bookmark lists populate the bookmarked entity and selected counts

## Collection Inventory Snapshots

### Old system database collections

- burdens
- charts
- configs
- episode_complete
- episodes
- harm_categories
- harm_types
- interventions
- pain_levels
- patients
- production_systems
- rd_1_acute_or_chronic
- rd_1_estimative_type
- rd_1_pain_depth
- rd_1_pain_level
- rd_1_pain_texture
- rd_2_unit_of_measurement
- segments
- tracks
- users

Atlas dashboard reference:

- https://cloud.mongodb.com/v2/5eaafefb6db91028f3b39645#/explorer/5eaaffaa6db91028f3b39edd/paintrack

### Current production database collections

| Collection name    | Properties | Storage size | Documents | Avg. document size | Indexes | Total index size |
| ------------------ | ---------- | -----------: | --------: | -----------------: | ------: | ---------------: |
| aigenerateds       | -          |      4.10 kB |         0 |                0 B |       1 |          4.10 kB |
| discussions        | -          |      4.10 kB |         0 |                0 B |       1 |          4.10 kB |
| episodes           | -          |     69.63 kB |       381 |           108.00 B |       1 |         45.06 kB |
| episodes_bookmarks | -          |      4.10 kB |         0 |                0 B |       4 |         16.38 kB |
| justifications     | -          |     36.86 kB |        24 |           215.00 B |       1 |         36.86 kB |
| patients           | -          |     45.06 kB |        76 |           174.00 B |       1 |         36.86 kB |
| patients_bookmarks | -          |     36.86 kB |         1 |            76.00 B |       4 |        147.46 kB |
| prompts            | -          |      4.10 kB |         0 |                0 B |       1 |          4.10 kB |
| recovery_passwords | -          |     32.77 kB |         0 |                0 B |       3 |         98.30 kB |
| segments           | -          |    106.50 kB |      1.1K |           239.00 B |       1 |         61.44 kB |
| set_password_codes | -          |     36.86 kB |         1 |           191.00 B |       3 |        110.59 kB |
| tracks             | -          |     61.44 kB |       353 |           139.00 B |       1 |         45.06 kB |
| update_email_codes | -          |     20.48 kB |         1 |           155.00 B |       3 |         61.44 kB |
| users              | -          |     36.86 kB |        38 |           185.00 B |       2 |         73.73 kB |

Atlas dashboard reference:

- https://cloud.mongodb.com/v2/5eaafefb6db91028f3b39645#/explorer/5eaaffaa6db91028f3b39edd/newpaintrackappprod

### Current development database collections

| Collection name    | Properties | Storage size | Documents | Avg. document size | Indexes | Total index size |
| ------------------ | ---------- | -----------: | --------: | -----------------: | ------: | ---------------: |
| aigenerateds       | -          |     77.82 kB |        12 |            3.75 kB |       1 |         36.86 kB |
| discussions        | -          |     36.86 kB |         7 |           648.00 B |       1 |         36.86 kB |
| episodes           | -          |     45.06 kB |       110 |           130.00 B |       1 |         36.86 kB |
| episodes_bookmarks | -          |     20.48 kB |         3 |            76.00 B |       4 |         81.92 kB |
| justifications     | -          |     36.86 kB |        40 |           216.00 B |       1 |         36.86 kB |
| patients           | -          |     45.06 kB |        45 |           215.00 B |       1 |         36.86 kB |
| patients_bookmarks | -          |     24.58 kB |         1 |            76.00 B |       4 |         98.30 kB |
| prompts            | -          |    139.26 kB |        13 |           13.41 kB |       1 |         36.86 kB |
| recovery_passwords | -          |     36.86 kB |         5 |           191.00 B |       3 |        110.59 kB |
| segments           | -          |    110.59 kB |       680 |           276.00 B |       1 |         53.25 kB |
| set_password_codes | -          |     24.58 kB |         0 |                0 B |       3 |        110.59 kB |
| tracks             | -          |     45.06 kB |       173 |           143.00 B |       1 |         36.86 kB |
| update_email_codes | -          |     36.86 kB |         2 |           168.00 B |       3 |        110.59 kB |
| users              | -          |     36.86 kB |        29 |           214.00 B |       2 |         73.73 kB |

Atlas dashboard reference:

- https://cloud.mongodb.com/v2/5eaafefb6db91028f3b39645#/explorer/5eaaffaa6db91028f3b39edd/newpaintrackapp

## Architectural Caveats

### Incomplete cascades

There is no single delete path that fully removes all descendants from patient down to justifications. Orphaned records are possible.

Examples:

- deleting a patient triggers episode cleanup, but episode cleanup can use `deleteMany` on tracks and skip the track use case that deletes segments
- deleting an episode can delete tracks without deleting segments
- deleting a track deletes segments, but segment justifications are not automatically deleted
- deleting prompts does not clearly clean AI-generated records

## Short Summary

Pain Track is organized around two generations of client-server repositories backed by MongoDB. The current domain model centers on user-owned patients, nested episodes, tracks, segments, and discussion threads, with bookmarks and auth-related helper records around the edges. The most important architectural reality is that relation integrity depends heavily on application code rather than hard guarantees from the database.