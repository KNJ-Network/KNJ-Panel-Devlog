# Phase 165 - The Box Was Too Small For Its Own Text

Text boxes on several pages had text sitting right at the edge of the field. Rather than patch one page, we audited every form field in the view templates and found that more than half set no padding at all.

## One rule instead of many files

There is no shared input component, so each form styles its own fields. We added a single base-layer rule giving text-entry fields a sensible default padding. Because utility classes always take priority over base styles, fields that already set their own padding are unaffected. The few fields with too little padding were widened by hand.

A comment containing a character sequence that closed the CSS comment early was caught immediately by the build and rewritten.

## Verifying

Full suite (2563/2563) and `pint --test`, then a visual check on the development server confirmed comfortable spacing.
