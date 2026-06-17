OVERVIEW
--------
This project provides a structured workflow for managing ArcGIS Experience
Builder (EXB) app templates hosted in Portal for ArcGIS Enterprise 11.3.

It is designed to run inside an ArcGIS Pro Notebook on a Staging environment.
All changes are reviewed and approved manually before being pushed to the
Portal. Promotion to Production is handled by separate elevation tooling
outside this project.

--------------------------------------------------------------------------------

PREREQUISITES
-------------
- ArcGIS Pro with active Portal connection to Staging environment
- Python environment via ArcGIS Pro (arcgis package included)
- Access to the Portal items you intend to manage

--------------------------------------------------------------------------------

SETUP
-----
1. Clone or copy this project into your ArcGIS Pro project directory.

2. Update the three path constants at the top of Cell 1 in the notebook
   to match your local directory:

       MANIFEST_PATH = r"C:\your\arcpro\project\exb_templates\manifest.json"
       TEMPLATES_DIR = r"C:\your\arcpro\project\exb_templates\templates"
       REVIEW_DIR    = r"C:\your\arcpro\project\exb_templates\review_workspace"

3. Confirm the folder structure exists. Create manually if needed:

       exb_templates/
       |-- templates/
       └-- review_workspace/

4. Open exb_template_factory.ipynb in ArcGIS Pro Notebooks.

5. Run Cells 1 through 5 to initialize the connection and load all functions.

--------------------------------------------------------------------------------

NOTEBOOK CELLS
--------------

Cell 1 - Imports and Connection
    Connects to the active Portal via GIS("pro") and loads path constants
    and manifest helper functions. Always run this cell first.

Cell 2 - pull_config(template_name, item_id)
    Pulls the Experience Builder config from Portal by Item ID.
    Saves a copy to templates/<template_name>/config.json and a working
    copy to review_workspace/working_config.json.
    Writes the Item ID and pull timestamp back to manifest.json.

Cell 3 - push_config(template_name)
    Reads the approved config from templates/<template_name>/config.json
    and pushes it to the Staging Portal item. Portal REST API performs
    native validation. Push result and timestamp are recorded in
    manifest.json.

Cell 4 - show_manifest()
    Prints a readable summary of all templates currently in manifest.json,
    including Item IDs, group assignments, and last pull/push timestamps.

Cell 5 - add_template(template_name, portal_group, notes)
    Adds a new template entry to manifest.json. Does not pull or push.
    Use this when registering a new template before its first pull.

--------------------------------------------------------------------------------

STANDARD WORKFLOW
-----------------

1. Open ArcGIS Pro and confirm you are signed into the Staging Portal.

2. Open exb_template_factory.ipynb and run Cells 1 through 5.

3. Pull the template you want to work on:

       pull_config("template_public_facing", "YOUR_ITEM_ID_HERE")

4. Hand review_workspace/working_config.json to your AI agent for review.
   Use the auditor prompt (see AI REVIEW GUIDELINES below).

5. Review each suggestion from the AI agent. Approve and apply changes
   manually to templates/<template_name>/config.json in your editor.
   Do not apply changes you have not reviewed.

6. Push the approved config back to Staging:

       push_config("template_public_facing")

7. Verify the changes look correct in the Staging Portal.

8. Use your standard elevation tooling to promote the validated Staging
   item to Production. This step occurs outside this project.

9. Commit your changes to Git after each completed review cycle.

--------------------------------------------------------------------------------

ADDING A NEW TEMPLATE
---------------------
1. Register the template in the manifest:

       add_template("template_field_ops", "EXB Templates - Field", "Field operations template")

2. Pull the config using its Portal Item ID:

       pull_config("template_field_ops", "YOUR_ITEM_ID_HERE")

3. Assign the item to the appropriate Portal Group through the Portal UI.
   Group membership is managed in Portal, not in this project.

--------------------------------------------------------------------------------

MANIFEST STRUCTURE
------------------
manifest.json tracks the state of each template locally. It is the audit
trail for this project and a reference for push operations.

    {
      "templates": {
        "template_public_facing": {
          "staging_item_id": "abc123",
          "portal_group": "EXB Templates - Public",
          "last_pulled": "2026-06-16T10:00:00+00:00",
          "last_pushed_staging": "2026-06-16T11:00:00+00:00",
          "notes": "WCAG AA compliant base template"
        }
      }
    }

Fields:
    staging_item_id      Portal Item ID, populated on first pull
    portal_group         Reference label only, managed in Portal UI
    last_pulled          UTC timestamp of last pull
    last_pushed_staging  UTC timestamp of last push to Staging
    notes                Free text for template description or status

--------------------------------------------------------------------------------

AI REVIEW GUIDELINES
--------------------
When handing working_config.json to an AI agent for review, use the
following prompt structure to keep output focused and safe:

    "You are acting as a Lead Developer and Accessibility Auditor.
    I am providing you with an ArcGIS Experience Builder config.json file.

    Please perform a review against the following criteria:

    [STRUCTURAL]
    - Every widget.id must be globally unique within the config
    - Every dataSourceId referenced in a widget must exist in root.datasources

    [ACCESSIBILITY - WCAG 2.1 AA]
    - Interactive widgets must have aria-label or aria-labelledby defined
    - Theme tokens must use var(--) references, not hardcoded color values

    [PERFORMANCE]
    - No duplicate datasource declarations pointing to the same service URL
    - Map widgets should not load excessive layers on initialization

    Output a list of violations only. Include the exact JSON path and a
    brief explanation for each issue. Do not rewrite the file."

Review each flagged item individually. Apply approved changes manually.
Do not use AI-generated output to overwrite files directly.

--------------------------------------------------------------------------------

ENVIRONMENT NOTES
-----------------
- This notebook targets Staging only. Do not run against Production.
- Portal connection is inherited from the active ArcGIS Pro session.
- No credentials are stored in this project. Authentication is handled
  by ArcGIS Pro.
- Both Staging and Production run ArcGIS Enterprise 11.3 in federated
  environments hosted on separate machines.
- Promotion from Staging to Production is handled outside this project
  by separate elevation tooling.

--------------------------------------------------------------------------------

KNOWN LIMITATIONS
-----------------
- push_config() relies on Portal REST API native validation. No additional
  schema validation is applied at the Python level by design.
- manifest.json is a local file and is not synced to Portal automatically.
- Portal Group membership is reference only in the manifest. Actual group
  assignment must be managed through the Portal UI.

--------------------------------------------------------------------------------

VERSION
-------
Initial version — June 2026
ArcGIS Enterprise 11.3 | Portal for ArcGIS | Experience Builder

================================================================================
