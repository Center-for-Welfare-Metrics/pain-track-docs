# Old Pain App Summary

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

## How the current project relates to it

The current project came from the effort to refactor and improve the old one.

That newer direction aimed to:

- preserve the core functionality that already existed
- improve structure and maintainability
- improve security
- create room for features that had been planned but not fully delivered before, including discussions

The discussions feature is partially implemented in the new Pain Track app, but it is still not complete.

Users can comment, which allows some level of discussion, but they still cannot suggest changes in a structured or effective way. In practice, collaboration is still limited because feedback happens mainly through comments rather than through a clearer suggestion-and-review workflow.

So the current project should be understood as the next stage after the old Pain Track app, not as a separate product with an unrelated origin.

## Planned in old docs but not available in the current project

Except for the gaps listed in this section, the rest of this document still applies to the current project at a high level. The current project remains a continuation of the same core product, with the same main functional purpose and most of the same core flows. The points below are the main places where the old documentation should be read as planned or legacy-oriented rather than as a description of what exists today.

- The forum implementation described in the old documentation was not delivered in the current project. The old plan described a public Pain-Tracks Forum where authors would submit Pain Episodes to a public repository, submitted episodes would become discussion spaces, and each segment would act as a thread context.
- The old plan also said that authors would be able to incorporate or not incorporate suggestions, and that dissatisfied participants could create their own Pain Episodes in response. That proposal-and-response workflow is not implemented in the current project.
- What exists today is much narrower. As documented in the current flow documentation, discussions can be opened in patient, episode, track, and segment contexts, and users can create discussions and add comments. However, this is still a comment-section-like feature, not the planned forum workflow.
- The old documentation also defined an explicit database naming and reference-data convention, especially around `rd_*` prefixed reference-data collections and a more formalized naming scheme. That legacy convention is not carried into the current project as a documented or visible reference-data layer in the same way. Some underscore-based field naming still appears in the current data model, but the old `rd_*` structure should not be treated as an implemented part of the current system.

## Short summary

The old Pain Track app was a working implementation of the original core system, but it is now outdated.

Most of the main functionality was implemented, except for the discussions feature. The current project was created as a refactor and improvement of that old version, while keeping the same core idea and extending it in a more secure and maintainable way. In the new app, discussions exist only partially: users can comment, but they still cannot suggest changes in a robust collaborative way.