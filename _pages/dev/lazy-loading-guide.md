---
title: Lazy Loading
feature:
- name: Lazy Loading of CMS components
  spa_version: 2.0
  cx_version: n/a
  anchor: "#lazy-loading-of-cms-components"
- name: Lazy Loading of Modules
  spa_version: 2.1
  cx_version: n/a
  anchor: "#lazy-loading-of-modules"
---

Lazy loading, also known as code splitting, lets you divide your JavaScript code into multiple chunks. The result is that you do not have to load all the JavaScript of the full application when a user accesses the first page. Instead, only the chunks that are required for the given page are loaded. While navigating the storefront, additional chunks are loaded when needed.

Such an approach can substantially improve "Time To Interactive", especially in the case of complex web applications being accessed by low-end mobile devices.

***

**Table of Contents**

- This will become a table of contents (this text will be scrapped).
{:toc}

***

## Spartacus Approach to Lazy Loading

Code splitting is a technique that has to be done at application build time. Code splitting provided by Angular is typically route-based, which means there is a chunk for the landing page, another chunk for the product page, and so on. Since Spartacus is mostly CMS driven, the actual application logic for each route cannot be decided at build time. Business users will eventually alter the page structure by introducing or removing components. This is why an alternative approach to lazy loading is required, which Spartacus provides in the following ways:

- Lazy loading of CMS components
- CMS-driven lazy loading of feature modules

## General Concepts

The following sections offer some important information about how lazing loading works in Spartacus.

### Defining Dynamic Imports Only in the Main Application

Dynamic imports, a technique that is used to facilitate lazy loading and that also allows code splitting, can only be used in the main application. It is not possible to define dynamic imports in the prebuilt libraries.

This is an unfortunate limitation, which results in some application code that must be added by customers. Although the amount of custom code is limited to the bare minimum, we'll add a feature in a future version of the schematics library to automatically add lazy-loaded modules.

### Avoiding Static Imports for Lazy-Loaded Code

To make code spitting possible, your static JavaScript code (the main app bundle) should not have any static imports to code that you want to lazy load. The builder will notice that the code is already included, and as a result, will not generate a separate chunk for it. This is especially important in the case of importing symbols from libraries.

At the time of writing (Angular 9 and Angular 10), mixing static imports with dynamic imports for the same library entry point, even for distinct symbols, will break lazy loading and tree shaking for this library entry point. If you were to do this, it would statically include the entire entry point in the build. For this reason, it is highly recommended that you create specific entry point for the code that has to be loaded statically, and separate entry points for the code that can be loaded lazily.

### Configuration in Lazy-Loaded Modules

{% capture version_note %}
{{ site.version_note_part1a }} 3.0 {{ site.version_note_part2 }}
{% endcapture %}

{% include docs/feature_version.html content=version_note %}

If additional configuration is provided inside the lazy-loaded module, Spartacus merges it into the global application configuration to support lazy loading scenarios for existing components and services. In most cases, especially when lazy-loaded modules provide mostly default configurations, this works reliably. However, it can cause problems if it is overused, especially when two modules provide different configurations for the same part of the config. Scenarios such as these can be fixed by providing the necessary overrides in the main app.

This merging functionality is made possible by a compatibility mechanism that is enabled by default, but you can disable it with the `disableConfigUpdates` feature flag. If you are developing new modules that have to hook into configurations from lazy-loaded modules, you should use the `ConfigurationService.unifiedConfig$` instead. This functionality is described in the following section.

#### Unified Configuration

Unified configuration provides a way to get a global configuration that includes both the root configuration and the configuration from already-loaded lazy-loaded modules.

The `ConfigurationService.unifiedConfig$` exposes the unified configuration as an observable that emits the new configuration each time it changes. This happens, for example, every time a lazy-loaded module with a provided configuration is loaded and instantiated.  

All configuration parts are merged in a strict order, where the actual configuration always overrides the default one, and the configuration defined in the root module (that is, the app shell) has precedence.

The following is an example that shows the order in which different configurations provided in the root app and in two lazy-loaded modules are merged, where each subsequent item in the list can override the previous one:

- default root configuration
- default configuration from lazy-loaded module 1
- default configuration from lazy-loaded module 2
- configuration from lazy-loaded module 1
- configuration from lazy-loaded module 2
- root configuration (always takes precedence)  

### Providers in Lazy-Loaded Modules

Injection tokens provided in lazy-loaded modules are not visible to services provided in the root application. This applies especially to multi-provided tokens, such as `HttpInterceptors`, various handlers, and so on.

To mitigate this drawback, some Spartacus features, such as the `PageMetaService` (which consumes `PageMetaResolver` tokens), or the `ConverterService` (which mostly consumes adapter serializers and normalizers), use the unified injector under the hood. In doing so, they have access to lazy-loaded tokens and can leverage them for global features.

For mechanisms that do not rely on the unified injector (for example, functionality from most non-Spartacus libraries, such as the core Angular libraries), it is recommended that you always eager load modules with these kinds of tokens.

For more information about eager loading, see the [Angular documentation glossary](https://angular.io/guide/glossary#eager-loading).

#### Unified Injector

The unified injector provides a way to inject a token, or multi-provided tokens, while taking into account the root injector and the injectors from lazy-loaded features. The injector exposes an observable that emits a new set of injectables for a specified token each time the status of the unified injector changes.

### Avoiding Importing the HttpClientModule in Your Lazy-Loaded Modules

In general, the `HttpClientModule` should be imported in the root application, not in the library. For example, if you import it inside a lazy-loaded library, all injectors from the root library will be invisible to HTTP calls originating from the lazy-loaded module.

Although technically it is possible to import the `HttpClientModule` in the library, in most cases it is not something that is expected, and it might cause errors that are difficult to explain, so please keep this in mind.

## Lazy Loading of CMS components

{% capture version_note %}
{{ site.version_note_part1 }} 2.0 {{ site.version_note_part2 }}
{% endcapture %}

{% include docs/feature_version.html content=version_note %}

### Configuration of Lazy Loading CMS Components

Lazy loading of CMS code is achieved by specifying a dynamic import in place of a statically-referenced components class in the CMS Mapping configuration. The following is an example:

```typescript
{
  cmsComponents: {
    SimpleResponsiveBannerComponent: {
      component: () => import('./lazy/lazy.component').then(m => m.LazyComponent)
    }
  }
}
```

For more information on CMS mapping, see [Customizing CMS Components]({{ site.baseurl }}{% link _pages/dev/components/customizing-cms-components.md %}).

### Technical Details

Support for dynamic imports in CMS component mapping is implemented using customizable component handlers (specifically, the `LazyComponentHandler`).

It is possible to extend this handler to customize its behavior, to add special hooks, or different triggers, or to implement a completely new handler that can optionally reuse existing ones.

## Lazy Loading of Modules

{% capture version_note %}
{{ site.version_note_part1 }} 2.1 {{ site.version_note_part2 }}
{% endcapture %}

{% include docs/feature_version.html content=version_note %}

CMS-driven lazy loading of feature modules allows the following:

- Lazy loading not only component code, but also the core part (including NgRx state)
- Loading a feature only once, when first needed
- Providing shared, lazy-loaded dependency modules

Lazy loading of the feature module is triggered by the CMS requesting a component when the implementation is covered by the relevant feature configuration.

### Configuration of Lazy-Loaded Modules

Configuration of the feature involves the following aspects:

- The dynamic import of the feature module (which has to be defined in main application)
- The information about what CMS components are covered by a specific feature (which can become part of the library and imported statically). This information is provided as an array of strings under the `cmsComponents` key. The following is an example:

  ```typescript
  {
    featureModules: {
      organization: {
        module: () =>
          import('@spartacus/my-account/organization').then(
            (m) => m.OrganizationModule
          ),
        cmsComponents: [
          'OrderApprovalListComponent',
          'ManageBudgetsListComponent',
          'ManageCostCentersListComponent',
          'ManagePermissionsListComponent',
          'ManageUnitsListComponent',
          'ManageUserGroupsListComponent',
          'ManageUsersListComponent',
        ],
      },
    },
  }
  ```

### Component Mapping Configuration in Lazy-Loaded Modules

Default CMS mapping configuration in lazy-loaded modules should be defined in exactly the same fashion as in statically-imported ones.

Spartacus is able to extract a CMS component mapping configuration from a lazy-loaded feature and use it to resolve component classes and factories covered by the feature. That is why it is possible and recommended to use the standard way of providing the default CMS mapping configuration inside a lazy-loaded module. As a result, the exact same module and library entry point can be imported dynamically or statically, depending on the needs, and it is still possible to overwrite a lazy-loaded CMS configuration from the application level by providing configuration overrides in the application.

### Defining Shared-Dependency Modules

It is possible to extract some logic to a shared, lazy-loaded module that can be defined as a lazy-loaded dependency for a feature module by providing an array of dynamic imports in the `dependencies` property of the feature configuration. The following is an example:  

```typescript
{
  featureModules: {
    organization: {
      module: () =>
        import('@spartacus/my-account/organization').then(
          (m) => m.OrganizationModule
        ),
      dependencies: [
        () =>
          import('@spartacus/storefinder/core').then(
            (m) => m.OrganizationModule
          ),
        // ,,
      ],
    },
  },
}
```

Such unnamed dependency modules are instantiated only once, when lazy loading the first feature that depends on it. Its providers contribute to the combined injector that is passed to the feature module, and as a result, all feature services and components have access to the services provided by the dependency modules.  

### Combined Injector

Any lazy-loaded module can inject (that is, have access to) services and tokens from the root application injector and the dependency modules injectors. This is possible because of the `CombinedInjector` that is created each time the feature module with dependencies is instantiated.

When a CMS component that is covered by a lazy-loaded module is instantiated, it can inject (that is, have access to) services from the following:

- The `ModuleInjector` hierarchy, starting from the feature module injector, and including dependency modules and the root injector
- The `ElementInjector` hierarchy, which is created implicitly at each DOM element

## Preparing Libraries to Work with Lazy Loading

### Providing Fine-Grained Entry Points in Your Library

Mixing static and dynamic imports from the same entry points breaks lazy loading and affects tree shaking, so any library that will be used directly in dynamic imports should expose fine-grained secondary entry points to optimize code splitting.

As a convention, Spartacus exposes root entry points for features, such as `@spartacus/orgainzation/administration/root`. This type of entry point contains all of the code that should not or cannot be lazy loaded. Modules from the root entry point should be imported statically in the root application, which means they will be eager loaded and will be available in the main application chunk.

For more information about support for secondary entry points in the Angular libraries, see [Secondary Entry Points](https://github.com/ng-packagr/ng-packagr/blob/master/docs/secondary-entrypoints.md) in the ng-packagr documentation on GitHub.

### Separating Static Code from Lazy-Loaded Code

When you work with Angular Dependency Injection, the list of providers in the injector should not change after the injector is initialized. This paradigm specifically applies to any multi-provided tokens, to handlers, and especially to any Angular-native multi-provided tokens, such as `HTTP_INTERCEPTOR`, `APP_INITIALIZER`, and so on.
  
The result is that any multi-provided token in a lazy-loaded module will not be visible to modules and services that are provided in the root, or in other lazy-loaded chunks, with the exception of multi-provided tokens that are injected using the unified injector.

Some Spartacus features, such as `PageMetaService` or `ConverterService`, use the `UnifiedInjector` to be aware of tokens that can be lazy loaded, so that the global logic (such as SEO features) can work reliably even if the logic is lazy loaded with the feature. For example, the store locator page meta resolver can be lazy loaded with the store locator feature.

The Spartacus configuration, which is also defined by providing configuration chunks, is treated a bit differently because of a compatibility mechanism that contributes configurations from lazy loading features to the global configuration. This mechanism can be disabled by a feature flag, and will be turned off by default in the future, in favor of the unified configuration feature.

If there are issues with lazy-loaded providers not being visible by the root services, it is always possible to include this kind of code in a statically-linked module that is available upfront. It is recommended to create a separate entry point in your library (by convention, named `root`, such as `my-library/root`) that contains minimal code, that will be included in the main bundle, and that will be available from the beginning.

### Wrapping Library Code in a Lazy-Loaded Module

To address some specific customizations on top of a lazy-loaded library, it is possible to wrap static imports from one or more libraries into a single module that will be imported dynamically instead. This technique can also be an effective optimization strategy, because the builder will be able to use tree shaking for all of the static imports inside the module.
