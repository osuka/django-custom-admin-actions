# Django custom actions for Model Admin forms

## Intro

Simple django app that extends the standard Change admin forms
to allow an extra bar of buttons with custom actions.

This is similar to how one can extend the List pages with
custom actions.

The extra button bar is shown only when there are actions.

Actions can be defined per object.

By default the button bar uses the same style as the normal bar but with a white background and a bigger font.

## Basics

- Extend from `custom_admin_actions.admin.CustomActionsModelAdmin`
- Implement `get_custom_admin_actions`
- Implement `custom_action_called`
- Optionally define overrides for `custom_admin_actions/css/custom_admin_actions.css`

## Internal architecture

Extends ModelAdmin, overrides `render_change_form` to add a list of actions to the context and overrides `response_change` to call methods to process the actions if submitted.

Provides an extension of the admin's default change_form and submit_line templates.

## Installation and usage

The package is not (as of yet) published anywhere so just
clone it into your project, the license allows you to do
whatever you want (BSD-3 clause).

Add it to your apps:

```python
INSTALLED_APPS += [
    "custom_admin_actions",  # additional row of actions for ModelAdmins
    ]
```

Make your ModelAdmin extend and declare a function to define a function
called `get_custom_admin_actions` that returns a dict declaring the
supported action codes and their corresponding labels:

```python
from custom_admin_actions.admin import CustomActionsModelAdmin

...

class SubmissionAdmin(SavesOwnerMixin, CustomActionsModelAdmin):
    ...
    def get_custom_admin_actions(
        self, request, context, add=False, change=False, form_url="", obj=None,
    ):
        if obj and obj.status == obj.Statuses.NEW:
            return {"start_moderation": "Move to Moderation", "reject": "Reject"}
        return {}

```

> Tip: you can use the request and context objects to hide/show actions

Now you can test the app and you'll already see the bar
![Image showing additional bar of buttons](./doc/submit_line_example.png)

Now, add a new function `custom_action_called` to the model admin page to process the actual button presses.

This function will receive the code (key) of the button that was pressed and then you can either return nothing to have the normal action after a change (go back to the list), or any other django http response. Example:

```python
    def custom_action_called(self, request, custom_action_code, obj=None):
        if custom_action_code == "start_moderation":
            moderated_submission = obj.move_to_moderation()
            # we'll redirect to the change page of a new object created from the current one
            redirect_url = reverse(
                "admin:%s_%s_change" % (models.ModeratedSubmission._meta.app_label, models.ModeratedSubmission._meta.model_name,), args=(moderated_submission.pk,), current_app=admin.site.name,
            )
            return HttpResponseRedirect(redirect_url)

        elif custom_action_code == "reject":
            obj.status = obj.Statuses.REJECTED
            obj.save()
            return None
```
