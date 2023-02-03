---
layout: classic-docs
title: "Scheduled Pipelines"
short-title: "Scheduled Pipelines"
description: "Learn about scheduled pipelines for your CircleCI projects."
contentTags:
  platform:
  - Cloud
---

**Scheduled pipelines are currently available for GitHub and Bitbucket VCS users.** Scheduled pipelines allow you to trigger pipelines periodically based on a schedule. Scheduled pipelines retain all the features of pipelines:

- Control the actor (yourself, or the scheduling system) associated with the pipeline, which can enable the use of [restricted contexts](/docs/contexts/#project-restrictions).
- Use [dynamic config](/docs/dynamic-config) via setup workflows.
- Modify the schedule without having to edit `.circleci/config.yml`.
- Take advantage of [auto-cancelling](/docs/skip-build/#auto-cancelling).
- Specify [pipeline parameters](/docs/pipeline-variables/#pipeline-parameters-in-configuration) associated with a schedule.
- Manage common schedules, for example, across workflows.

Scheduled pipelines are configured through the API, or through the project settings in the CircleCI web app.

A scheduled pipeline can only be configured for one branch. If you need to schedule for two branches, you would need to set up two schedules.
{: class="alert alert-info"}

## Introduction
{: #introduction }

Scheduled pipelines allow you to trigger pipelines periodically based on a schedule, from either the CircleCI web app or API. Schedules can range from daily, weekly, monthly, or on a very specific timetable. To set up basic scheduled pipelines, you do not need any extra configuration in your `.circleci/config.yml` file, however, more advanced usage of the feature will require extra `.circleci/config.yml` configuration (for example, workflow filtering, or using parameters).

Pipeline parameters are typed pipeline variables in the form of a string, integer, or boolean. Adding a parameter to a scheduled pipeline can be done in the web app in the triggers form while setting up a schedule. Any parameters set up in this manner must be added to your configuration file using the `parameters` key.

Scheduled pipelines are set to run by an "actor", either the CircleCI scheduling system, or a specific user (for example, yourself). The scheduling actor is important to consider if making use of restricted contexts in workflows. If the user (actor) running the workflow does not have access to the context, the workflow will fail with the `Unauthorized` status.

You can find a basic how-to guide on the [Set a nightly scheduled pipeline](/docs/set-a-nightly-scheduled-pipeline) page, and more advanced examples on the [Schedule pipelines with multiple workflows](/docs/schedule-pipelines-with-multiple-workflows) pages.

## Get started with scheduled pipelines
{: #get-started-with-scheduled-pipelines }

To get started with scheduled pipelines, you have the option of using the Scheduled Pipelines Orb, the API, or using the CircleCI web app. Each method is described below.

### Use Scheduled Pipelines Orb
{: #use-scheduled-pipelines-orb }

The CircleCI Scheduled Pipelines Orb lets you easily manage a project's schedules. The orb uses the list of schedules you provide to create, update or delete schedules in your project's pipeline. As a result, you can easily manage all of your pipeline's schedules from a single `json` file and any changes will be automatically updated. 

1. In the CircleCI web app, have your CCI token ready or create a new token by following the steps on the [Managing API tokens](/docs/managing-api-tokens) page. Add an environment variable with `CIRCLE_TOKEN` as the key and your CCI token as the value. This can be done through **Project Settings > Environment Variables** or added to a `context` in **Organization Settings > Contexts**. 
2. Create a `json` file with a list of your schedules in any directory of your project's repository. The list is a `json` object with an array of `schedules`. Each element in the array is a valid schedule payload for a `POST` request to the CircleCI Schedule's API. You can have a maximum of 50 schedules. You can find more details about the payload [schema here](https://circleci.com/docs/api/v2/index.html#operation/createSchedule). Here's an example:

```json
{
    "schedules": [{
        "name": "string",
        "timetable": {
            "per-hour": 0,
            "hours-of-day": [
            0
            ],
            "days-of-week": [
            "TUE"
            ],
            "days-of-month": [
            0
            ],
            "months": [
            "MAR"
            ]
        },
        "attribution-actor": "current",
        "parameters": {
            "deploy_prod": true,
            "branch": "feature/design-new-api"
        },
        "description": "string"
        }
    ]
}
```
*NOTE:* In the timetable key, `days-of-week` and `days-of-month` are mutually exclusive and cannot be used together.
3. In your project's `.circleci/config.yml` file, import the `scheduled-pipelines` orb with the `orb` parameter. Add the `scheduled-pipelines/update-schedules` job to your workflow. The job requires the `schedule-path` parameter, which is the file path to the `json` file containing your schedules. If you added your CCI token to a context, you will also need to provide the name of the context with the `context` parameter. Here's an example: 

```yml
description: >
  This is a simple example of updating pipeline schedules using the update-schedules job.
usage:
  version: 2.1
  orbs:
    scheduled-pipelines: circleci/scheduled-pipelines@1.0
  workflows:
    update-pipeline-schedules:
      jobs:
        - orb-scheduled-pipelines/update-schedules:
            schedule-path: ./path/to/schedules.json
            context: "context-with-access-token"
```

### Use project settings in the web app
{: #use-project-settings }

1. In the CircleCI web app, navigate to **Projects** in the sidebar, then click the ellipsis (...) next to your project and click on **Project Settings**. You can also find the **Project Settings** button on each project's landing page.
2. Navigate to **Triggers**.
3. To create a new schedule, click **Add Trigger**.
4. Define the new schedule by filling out the form, then click **Save Trigger**.

The form also provides the option of adding [pipeline parameters](/docs/pipeline-variables/), which are typed pipeline variables that you declare at the top level of a configuration.

If you would like to manage common schedules for multiple workflows, you will need to manually set this in your `.circleci/config.yml` file. See the [Schedule pipelines with multiple workflows](/docs/schedule-pipelines-with-multiple-workflows) page for examples.

### Use the API
{: #use-the-api }

If your project has no scheduled workflows, and you would like to try out scheduled pipelines:

1. Have your CCI token ready, or create a new token by following the steps on the [Managing API tokens](/docs/managing-api-tokens) page.
2. Create a new schedule [using the API](https://circleci.com/docs/api/v2/index.html#operation/createSchedule). For example:

```shell
curl --location --request POST "https://circleci.com/api/v2/project/<project-slug>/schedule" \
--header "circle-token: <PERSONAL_API_KEY>" \
--header "Content-Type: application/json" \
--data-raw '{
    "name": "my schedule name",
    "description": "some description",
    "attribution-actor": "system",
    "parameters": {
      "branch": "main"
      <additional pipeline parameters can be added here>
    },
    "timetable": {
        "per-hour": 3,
        "hours-of-day": [1,15],
        "days-of-week": ["MON", "WED"]
    }
}'
```

For additional information, refer to the **Schedule** section under the [API v2 docs](https://circleci.com/docs/api/v2).

## Migrate scheduled workflows to scheduled pipelines
{: #migrate-scheduled-workflows-to-scheduled-pipelines }

If you have existing scheduled workflows you need to migrate to scheduled pipelines, use the [Scheduled pipelines migration](/docs/migrate-scheduled-workflows-to-scheduled-pipelines) guide.

## Scheduled pipelines video tutorial
{: #scheduled-pipelines-video-tutorial }

The video offers a short tutorial for the following scenarios:

- Set a schedule in the web app
- Set a schedule with the API
- Migrate from scheduled workflows to scheduled pipelines

<div class="video-wrapper">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/x3ruGpx6SEI" title="Scheduled pipelines tutorial" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>

For the documentation for these scenarios, visit the following pages:
- [Set a nightly scheduled pipeline](/docs/set-a-nightly-scheduled-pipeline)
- [Schedule pipelines with multiple workflows](/docs/schedule-pipelines-with-multiple-workflows)

## FAQs
{: #scheduled-pipelines-faqs }

{% include snippets/faq/scheduled-pipelines-faq-snip.md %}

## Next steps
{: #next-steps }

- [Migrate scheduled workflows to scheduled pipelines](/docs/migrate-scheduled-workflows-to-scheduled-pipelines)
- [Schedule pipelines with multiple workflows](/docs/schedule-pipelines-with-multiple-workflows)
- [Set a nightly scheduled pipeline](/docs/set-a-nightly-scheduled-pipeline)
