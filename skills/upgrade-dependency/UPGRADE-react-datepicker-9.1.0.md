# Upgrade: react-datepicker → 9.1.0

## Summary

**Needs code changes (low risk).** rails-server is on react-datepicker 7.6.0; upgrading to 9.1.0 is supported by current usage. The same locale API (`getDefaultLocale`, `registerLocale`, `setDefaultLocale`) and DatePicker props are used in ui-design-system on 9.1.0, so the deprecated in-repo `DateTimePicker` should work after the bump. Run tests, update snapshots if the DOM changed, and do a quick visual check of date/time picker UIs.

## Current vs target

- Current: 7.6.0
- Target: 9.1.0
- Ecosystem: npm/yarn (rails-server)

**Note:** ui-design-system is already on react-datepicker ^9.1.0. This report covers upgrading the **rails-server** direct dependency only.

## Release notes summary

(Sourced from [releases page 1](https://github.com/Hacker0x01/react-datepicker/releases) and [page 2](https://github.com/Hacker0x01/react-datepicker/releases?page=2).)

- **Breaking changes:** v8.0.0 (31 Jan): fix for inconsistent/broken behavior in `parseDate` (#5036). v9.1.0 restores date parsing behavior that was accidentally removed in v8.0.0, so 7.6.0→9.1.0 gets the restored behavior.
- **Deprecations:** None called out in reviewed notes.
- **Migration / upgrade:** Bump version; no migration steps required for current API usage.
- **Security:** None mentioned.

### v9.1.0 (19 Dec)

- Restores date parsing behavior accidentally removed in v8.0.0; restores v8 default styles.
- New: `onClickOutside` can prevent calendar from closing via `event.preventDefault()`.
- Fixes: TypeScript props, empty `:global` selector (Lightning CSS), extra wrapper div with `withPortal`, Safari auto-translate, crash when date props are strings, base styles for month/year select dropdowns, clear selection when masked input cleared on blur, native Date fallback when `strictParsing` is false.

### v9.0.0 (major)

- New: Timezone support (`timeZone` prop, optional `date-fns-tz` peer); `showTimeSelect` / `showTimeInput` with `selectsRange`; props `popperTargetRef`, `monthHeaderPosition`, `renderCustomDayName`, `formatMultipleDates`, `aria-label`.
- Fixes: Calendar view when typing partial dates, when `selected`/`startDate` change programmatically; time picker height; month view jump in range mode; accessibility and TypeScript improvements.

### v8.0.0 (31 Jan) – breaking

- **Breaking:** fix for inconsistent/broken behavior in `parseDate` (#5036).
- Other: Upgrade to React 19, TypeScript linting for React 19, click outside within Shadow DOM, date-fns upgraded to v4.1.0.
- Full Changelog: v7.6.0...v8.0.0.

### v8.1.0–8.10.0

- Various fixes: input value refresh, CalendarIconProps optional, re-focus when open state controlled, containerRef propagation, etc.

### v7.6.0 (current)

- Fixes: Tab switch, "previous month" button, types, SCSS @use, week number highlight, Enter-after-delete custom input; React ^19 peer dep. Full Changelog: v7.5.0...v7.6.0.

## Affected usage

- `app/javascript/common/deprecated/form/date_time_picker/date_time_picker.jsx:7-11` – Imports DatePicker, getDefaultLocale, registerLocale, setDefaultLocale; uses DatePicker with selected, locale, dateFormat, showTimeSelect, popperProps, onCalendarClose, onChange.
- `app/javascript/packs/common.js:9` – Imports react-datepicker/dist/react-datepicker.css.
- `app/javascript/common/deprecated/form/date_time_picker/date_time_picker.css` – Overrides .react-datepicker__* classes (header, day-names, day--selected, time, etc.).
- `app/javascript/researcher/project_workspace/pages/research_design/activity/components/unmoderated_form_section/task_deadline.module.css:18` – Targets :global(.react-datepicker-popper).
- `app/javascript/researcher/account_availability/settings/settings.module.css:33` – Targets .react-datepicker__tab-loop.
- `cypress/support/commands/participant_onboard.js:25-27` – Uses .react-datepicker__month-select, .react-datepicker__year-select, .react-datepicker__day--008.
- `app/javascript/components/filters_popover/filter_types/spec/__snapshots__/date_filter_input_spec.tsx.snap` – Snapshots contain react-datepicker-wrapper, react-datepicker__input-container.

## Safety assessment

**Needs code changes (low risk)** – Upgrade is feasible with small follow-up. Locale API and core DatePicker props remain supported (ui-design-system uses same pattern on 9.1.0). No code changes required for date_time_picker.jsx; if v9 changed class names or DOM, Cypress selectors or Jest snapshots may need updates after upgrade.

## Upgrade steps

1. Edit package.json: change "react-datepicker": "^7.6.0" to "react-datepicker": "^9.1.0".
2. Run: yarn install
3. Apply code changes below only if tests or Cypress fail (see Regression test plan).

## Code changes

- None required for this upgrade. If date_filter_input_spec snapshot fails, run: yarn test -u app/javascript/components/filters_popover/filter_types/spec/date_filter_input_spec.tsx. If Cypress participant onboard fails, update selectors in cypress/support/commands/participant_onboard.js to match v9 DOM.

## Regression test plan

- **Existing tests:** yarn test (especially date_time_picker_spec.jsx, date_filter_input_spec.tsx). If snapshot diffs: yarn test -u app/javascript/components/filters_popover/filter_types/spec/date_filter_input_spec.tsx
- **Areas to verify:** Task deadline picker, hub date filter, date filter in filters popover, participant profile date attribute input, date answer input (surveys), participant onboard flow (Cypress month/year/day selects).
- **Manual QA:** Open date/time pickers (with and without time, with dropdowns); confirm layout, selection, closing. If using withPortal, confirm no layout regressions.
- **New tests:** Only if regressions or new behavior to lock in.

## Open questions / gaps

- If you rely on withPortal or custom popper behavior, double-check after upgrade (v9.1.0 changes wrapper structure).
