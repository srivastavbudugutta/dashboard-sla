**Status Update on 10/31/2023**

- Modified package.json to include Vue 3 compatible versions of plugins and dependencies. This ensures that all required libraries are aligned with Vue 3’s ecosystem.

-  Updated vite.config.js by integrating plugins that are specifically designed to work with Vue 3. These changes enable Vite to correctly handle Vue 3 features and optimizations during development and build processes.

**Status Update on 11/1/2023**

-  Refactored main.js to separate the app instance creation from mounting. This change provides greater flexibility for future configurations of the app instance.

**Status Update on 11/2/2023**

-  Introduced a `<header>` element and adjusted the margin to accommodate mobile device dimensions, enhancing both aesthetics and functionality. Used a `<header>` instead of non-semantic `<div>` in table.vue to add semantic meaning to the navigation bar.

- Added a `<div>` element as a container for the main table design to adhere to proper HTML structure within Vue components. This maintains the intended styles and overflow behavior and is configured to support the pagination design.

**Status Update on 11/3/2023**

Refactor `table.vue` to use Vue 3 Composition API

- Transitioned `table.vue` from Options API to Composition API for improved logical grouping and reactivity management.
- Encapsulated pagination logic within composable functions for enhanced maintainability.
- Defined reactive states using `ref` and computed states with `computed` for clearer reactivity tracking.
- Improved component readability and maintainability by organizing related functionalities together.

**Status Update on 11/4/2023**

- Completed the refactoring of the Table.vue component to improve code maintainability and readability.
- Integrated the Composition API for clearer reactivity tracking and better organization of logic.
- Implemented pagination functionality to handle large data sets efficiently which enhances user experience.
- Updated styling and added dynamic classes for status rows to improve visual feedback to the user.
- Conducted thorough testing to ensure all features are working as expected post-refactoring.

