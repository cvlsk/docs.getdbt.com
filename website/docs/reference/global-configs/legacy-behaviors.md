---
title: "Legacy behaviors"
id: "legacy-behaviors"
sidebar: "Legacy behaviors"
---

Most flags exist to configure runtime behaviors with multiple valid choices. The right choice may vary based on the environment, user preference, or the specific invocation.

Another category of flags provides existing projects with a migration window for runtime behaviors that are changing in newer releases of dbt. These flags help us achieve a balance between these goals, which can otherwise be in tension, by:
- Providing a better, more sensible, and more consistent default behavior for new users/projects.
- Providing a migration window for existing users/projects &mdash; nothing changes overnight without warning.
- Providing maintainability of dbt software. Every fork in behavior requires additional testing & cognitive overhead that slows future development. These flags exist to facilitate migration from "current" to "better," not to stick around forever.

These flags go through three phases of development:
1. **Introduction (Disabled by Default):** The logic is added within dbt to support both 'old' + 'new' behaviours. The 'new' behavior is gated by behind a flag, which is disabled by default, preserving the old behavior.
2. **Maturity (Enabled by Default):** The default value of the flag is switched, from `false` to `true`, enabling the new behavior by default. Users may still opt out of the 'new' behavior, preserving the 'old' behavior, by setting the flag to `false` in their projects. They may see deprecation warnings when they do so.
3. **Removal (Generally Enabled):** The flag, having been marked in advance for deprecation, is removed. The logic to support the 'old' behaviour is removed from dbt codebases. Our intent is to support most flags indefinitely, but we are not committed to supporting them forever. If we do choose to remove a flag, we will offer significant advance notice.

## Behavior Change Flags

These flags can _only_ be set in the `flags` dictionary in `dbt_project.yml`. They configure behaviors closely tied to project code, and as such they should be defined in version control and modified via pull/merge request, with the same testing and peer review.

The current flags are shown below, along with their current default values in the latest versions of dbt Cloud + dbt Core. To opt out of all upcoming behavior changes, set these values to `False`. You will continue to see deprecation warnings for each, which you may optionally silence via `warn_error_options.silence` [TODO: link].

<File name='dbt_project.yml'>

```yml
flags:
  require_explicit_package_overrides_for_builtin_materializations: False
  require_model_names_without_spaces: False
  source_freshness_run_project_hooks: False
```

</File>

| Flag                                                            | dbt Cloud: Intro | dbt Cloud: Maturity | dbt Core: Intro | dbt Core: Maturity | 
|-----------------------------------------------------------------|------------------|---------------------|-----------------|--------------------|
| require_explicit_package_overrides_for_builtin_materializations | 2024.04.xxx      | 2024.05.xxx         | 1.6.x, 1.7.x    | v1.8.0             |
| require_resource_names_without_spaces                           | 2024.04.xxx      | 2024.06.xxx         | 1.8.0           | v1.9.0             |
| source_freshness_run_project_hooks                              | 2024.03.xxx      | 2024.06.xxx         | 1.8.0           | v1.9.0             |

<sup>1</sup>dbt Cloud - "Keep on latest version"

</File>

### `require_explicit_package_overrides_for_builtin_materializations`

We have deprecated the behavior whereby installed packages could override built-in materializations without explicit opt-in from the end user. When this flag is set to `True`, a materialization defined in a package that matches the name of a built-in materialization will no longer be included in the search and resolution order.

The built-in materializations are `'view'`, `'table'`, `'incremental'`, `'materialized_view'` for models as well as `'test'`, `'unit'`, `'snapshot'`, `'seed'`, and `'clone'`.

Users can still explicitly override built-in materializations, in favor of a materialization defined in a package, by reimplementing the built-in materialization in their root project and wrapping the package implementation.

<File name='macros/materialization_view.sql'>

```sql
{% materialization view, snowflake %}
  {{ return(my_installed_package_name.materialization_view_snowflake()) }}
{% endmaterialization %}
```

In the future, we may extend the project-level `dispatch` configuration [TODO: link] to support a list of authorized packages for overriding built-in materialization.

</File>

### `require_resource_names_without_spaces`

The names of dbt resources (models, sources, etc) should contain letters, numbers, and underscores. We highly discourage the use of other characters. To that end, we have deprecated support for spaces in resource names. When this flag is set to the `True`, dbt will raise an exception (instead of a deprecation warning) if it detects a space in a resource name.

### `source_freshness_run_project_hooks`

This configures the `dbt source freshness` command to include "project hooks" (`on-run-start` / `on-run-end`, TODO add link) as part of their execution.

If you have specific project [`on-run-start` / `on-run-end`](/reference/project-configs/on-run-start-on-run-end) hooks that should not run before/after `source freshness` command, you can add a conditional check to those hooks:

<File name='dbt_project.yml'>

```yaml
on-run-start:
  - '{{ ... if flags.WHICH != 'freshness' }}'
```
</File>
