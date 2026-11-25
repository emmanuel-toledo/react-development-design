---
name: react-forms
description: Use when creating or reviewing a form in a React app, defining Zod schemas, using react-hook-form, handling submit, validation, server errors, dynamic fields, multi-step forms or unsaved-changes guards. Reuses shared form components; never creates inputs per form. Load this before adding a form.
---

# React Forms

Owns **how forms are built** in a React application. Load this before creating or reviewing a form.

## When to use

- Creating or reviewing a form component.
- Defining a Zod schema or an inferred form type.
- Handling submit, validation, server errors, dynamic fields or multi-step flows.
- Adding an unsaved-changes guard.
- Deciding where a form-related file belongs.

For feature structure, read `react-feature`. For layers and imports, read `react-architecture`. For shared components, read `react-shared`. For the library stack, read `react-stack`. For testing, read `react-testing`. For effects, read `react-effects`.

## Conventions

- Schema file: `features/<feature>/utils/<entity>.schema.ts`.
- Form type file: `features/<feature>/types/<entity>.form.type.ts` (inferred with `z.infer`).
- Form component: `features/<feature>/components/<Entity>Form.tsx`.
- No `schemas/` folder. Zod schemas live in `utils/`.

## Principles

1. **Reuse shared components.** Use `shared/components/forms` and `shared/components/ui` and shadcn components. Do not create input components per form. If a needed input does not exist in shared, add it to shared (with promotion approval) instead of inlining it in the feature.
2. **react-hook-form + Zod.** Use `zodResolver` from `@hookform/resolvers/zod`. Derive the form type with `z.infer`.
3. **One `useForm` per form** with explicit `defaultValues`.
4. **Submit goes through a React Query mutation.** Disable the submit button and show a spinner while pending; prevent double submit.
5. **Server errors:** map them to fields with `setError` and show a general toast (Sonner).
6. **Dynamic fields** with `useFieldArray` and stable keys.
7. **Multi-step (wizard) is optional:** state per step + validation per step; single submit at the end.
8. **Reset** with `reset()` or by changing the `key` when the entity changes.
9. **Async validation** (e.g. unique username) with debounce via RHF `validate`.
10. **Accessibility:** labels, `aria-invalid`, associated error messages, focus on first error.
11. **Messages via i18n keys** (react-i18next), not hardcoded strings.

## Unsaved-changes guard

If the form has unsaved data (`formState.isDirty === true`) and the user tries to leave, ask for confirmation.

- Hook: `shared/hooks/useUnsavedChanges`.
- Covers internal navigation (React Router `useBlocker`) and tab/browser close (`beforeunload`).
- Also applies to modals/drawers containing a form.
- Exception: after a successful submit, allow leaving without confirmation (reset dirty).
- The guard is **disabled while submitting** (`isSubmitting` / mutation pending) and **re-enabled if the submit fails**.
- Standard confirmation message (configurable per feature if needed).

## Anti-patterns (do not)

- Creating custom inputs per form when a shared one exists.
- Using `useState` for form values instead of RHF.
- Manually validating what Zod already covers.
- Fetching in `onSubmit` without a mutation.
- Hardcoded error messages when i18n is in use.
- Ignoring the unsaved-changes guard.
- Leaving the guard active during submit.

## Checklist

- [ ] Reuses components from `shared/components`.
- [ ] Schema in `utils/<entity>.schema.ts`; type in `types/<entity>.form.type.ts`.
- [ ] `zodResolver` + RHF with explicit `defaultValues`.
- [ ] Submit → mutation, loading/disabled, no double submit.
- [ ] Server errors → `setError` + toast.
- [ ] Dynamic fields with stable keys (if applicable).
- [ ] Async validation with debounce (if applicable).
- [ ] Accessibility (labels, `aria-invalid`, focus on first error).
- [ ] i18n for error messages.
- [ ] Unsaved-changes guard (navigation + tab close + modals), disabled during submit.
- [ ] Tests pass and coverage ≥ 90%.

## References

| Situation | Read |
| --- | --- |
| Feature structure and isolation | `react-feature` |
| Layer rules and imports | `react-architecture` |
| Shared components and shadcn location | `react-shared` |
| Library stack and versions | `react-stack` |
| Testing strategy and coverage | `react-testing` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Routing (paths, guards, params) | `react-routing` |