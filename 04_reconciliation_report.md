# Old Pain App Summary

> **Reading note (2026-09-06):** this reconciliation preserves the functional relationship between the two source generations. In this document, “current project” means the later/current-generation codebase; it does not mean that the application is currently active. Pain Track is intentionally paused.

The old Pain Track app was the first implemented version of the system.

It included the main core flows of the application and, for the most part, delivered what had originally been planned for that version. The main exception was the discussions feature, which was proposed but was not completed in the old app.

## What the old app included

The old version covered the main application behavior needed to use Pain Track in practice, including:

- authentication
- patient management
- episode creation
- tracks and segments
- general navigation and data entry flows

In that sense, the old app was a working implementation of the original core product idea.

## What was missing

The main missing piece was the discussions feature.

That feature was meant to allow more collaborative review of pain estimates, especially around segments and their justification, but it was not fully implemented in the old version.

## Current status of the old app

The old app is now outdated code.

It has not received updates for a long time, and it should be treated as a legacy system rather than an actively evolving product.

## How the current-generation source relates to it

The current-generation source came from the effort to refactor and improve the old one.

That newer direction aimed to:

- preserve the core functionality that already existed
- improve structure and maintainability
- improve security
- create room for features that had been planned but not fully delivered before, including discussions

The discussions feature is partially implemented in the later Pain Track source, but it is still not complete.

The source implements comments, which were intended to allow some discussion, but it does not implement structured change proposals. If the application is ever reactivated, that limitation would remain unless the feature is changed and tested.

So the current-generation source should be understood as the next stage after the old Pain Track app, not as a separate product with an unrelated origin.

## Planned in old docs but not implemented in the current-generation source

Except for the gaps listed in this section, the rest of this document still applies to the current-generation source at a high level. It remains a continuation of the same core product, with the same main functional purpose and most of the same represented flows. The points below are the main places where the old documentation should be read as planned or legacy-oriented rather than as a description of implemented source behavior.

- The forum implementation described in the old documentation was not delivered in the current project. The old plan described a public Pain-Tracks Forum where authors would submit Pain Episodes to a public repository, submitted episodes would become discussion spaces, and each segment would act as a thread context.
- The old plan also said that authors would be able to incorporate or not incorporate suggestions, and that dissatisfied participants could create their own Pain Episodes in response. That proposal-and-response workflow is not implemented in the current project.
- What the current-generation source implements is much narrower. It represents discussions in patient, episode, track, and segment contexts and provides comment creation, but not the planned forum workflow. This is a source-level description, not a current service-availability claim.
- The old documentation also defined an explicit database naming and reference-data convention, especially around `rd_*` prefixed reference-data collections and a more formalized naming scheme. That legacy convention is not carried into the current project as a documented or visible reference-data layer in the same way. Some underscore-based field naming still appears in the current data model, but the old `rd_*` structure should not be treated as an implemented part of the current system.

## Short summary

The old Pain Track app was a working implementation of the original core system, but it is now outdated.

Most of the main functionality was implemented, except for the discussions feature. The current-generation source was created as a refactor and improvement of that old version while keeping the same core idea. Its discussion code supports comments but not a robust suggestion-and-review workflow. None of these preserved behaviors should be treated as currently available while Pain Track is paused.
