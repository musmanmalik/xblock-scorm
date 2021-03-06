edx_xblock_scorm
=========================

XBlock to display SCORM content within the Open edX LMS.  Editable within Open edx Studio. Will save student state and report scores to the progress tab of the course.
Currently supports SCORM 1.2 standard, but not yet SCORM 2004.  It supports multi-SCO content packages.


# Installation

## Using Ansible

* Add to your `server-vars.yml` file this entry underneath the `EDXAPP_EXTRA_REQUIREMENTS` key

```
    - name: 'git+https://github.com/appsembler/edx_xblock_scorm@use_ssla_player#egg=scormxblock'
```

* Configure a SCORM player backend in the XBlock settings, under the `"ScormXBlock"`.  Currently you must provide the SSLA player by JCA Solutions.

```
EDXAPP_XBLOCK_SETTINGS:
  "ScormXBlock": {
    "SCORM_PLAYER_LOCAL_STORAGE_ROOT": "scormplayers",
    "SCORM_PLAYER_BACKENDS": {
      "ssla": {
        "name": "SSLA",
        "location": "/static/scorm/ssla/player.htm",
        "configuration": {}
      }
    },
    "SCORM_PKG_STORAGE_DIR": "scorms"
  }
```

Each backend is a key under `SCORM_PLAYER_BACKENDS` and should provide a `"name"` which will appear in the player dropdown in Studio, a `"location"` which is a path to the the player's default HTML page from the the web root, and optionally an additional key `"configuration"` storing JSON values which will override any JSON configuration keys used by the player (not yet fully implemented). Backend keys must not contain spaces or punctuation.

* Configure a SCORM package storage directory.  This will be the directory name underneath the default `MEDIA_ROOT` as specified in your Django settings, or the directory used for external storage (e.g., Amazon S3.  S3 storage will require using CloudFront or another means to serve S3 assets from the same protocol, domain, subdomain,and port to get around cross-domain issues).

* Configure Staff Debug Info: If you want staff debug info for SCORM XBlocks to be visible to instructors and other permitted staff, add this key to your `EDXAPP_XBLOCK_SETTINGS` key for 'ScormXBlock'.  Only the 'Delete Student State' action will work; 'Reset Student Attempts' and 'Rescore Student Submission' are not yet operable, but may be in the future.

```
"SCORM_DISPLAY_STAFF_DEBUG_INFO": true
```


# Server configuration

Nginx (or other front-end web server) must be configured to serve SCORM content. See the file [`docs/nginx_configuration.md`](docs/nginx_configuration.md) for edits that need to be made to your `/etc/nginx/sites-enabled/lms` and `/etc/nginx/sites-enabled/cms` files to serve your SCORM content.

# Usage
* In Studio, add `scormxblock` to the list of advanced modules in the advanced settings of a course.
* Add a 'scorm' component to your Unit.
* Specify a display name, an optional description, and choose a 'player' (IMPORTANT: currently only works using the commercial SSLA player from JCA Solutions.  Contact Appsembler if you would like assistance with this).
* Specify a weight for your SCORM content within your Open edX course.  For example, your course SCO may have an internal max. score of 100.  If you give your SCORM component a weight of 10 and a student scores 50 within the SCORM course, they will receive 5 points within Open edX.  SCORM components can be weighted as normal against other gradable Open edX components.
* Choose a display format (popup or iframe -- only use iframe if your SCORM content package itself creates a popup window) and display size.
* Upload a zip file containint your content package.  The `imsmanifest.xml` file must be at the root of the zipped package (i.e., make sure you don't have an additional directory at the root of the Zip archive which can handle if e.g., you select an entire folder and use Mac OS X's compress feature).
* Publish your content as usual.

# Translation (i18n)

This repo offers multiple make targets to automate the translation tasks.
Each make target will be explained below:

- `extract_translations`. Use [`i18n_tool` extract](https://github.com/edx/i18n-tools) to create `.po` files based on all the tagged strings in the python and javascript code.
- `compile_translations`. Use [`i18n_tool` generate](https://github.com/edx/i18n-tools) to create `.mo` compiled files.
- `detect_changed_source_translations`. Use [`i18n_tool` changed](https://github.com/edx/i18n-tools) to identify any updated translations.
- `validate_translations`. Compile translations and check the source translations haven't changed.

If you want to add a new language:
  1. Add language to `scormxblock/translations/config.yaml`
  2. Make sure all tagged strings have been extracted:
  ```bash
  make extract_translations
  ```
  3. Clone `en` directory to `scormxblock/translations/<lang_code>/` for example: `scormxblock/translations/fa_IR/`
  4. Make necessary changes to translation files headers. Make sure you have proper `Language` and `Plural-Forms` lines.
  5. When you finished your modification process, re-compile the translation messages.
  ```bash
  make compile_translations
  ```


