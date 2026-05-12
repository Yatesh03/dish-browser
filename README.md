# Meals App 🍽️

A Flutter application that lets users browse meal recipes organized by category, apply dietary filters, and save their favorite meals — all wrapped in a clean dark-themed UI.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Project Structure](#project-structure)
- [App Workflow](#app-workflow)
  - [1. App Entry Point](#1-app-entry-point)
  - [2. Tabs Screen (Home)](#2-tabs-screen-home)
  - [3. Categories Screen](#3-categories-screen)
  - [4. Meals Screen](#4-meals-screen)
  - [5. Meal Details Screen](#5-meal-details-screen)
  - [6. Filters Screen](#6-filters-screen)
  - [7. Favorites Tab](#7-favorites-tab)
  - [8. Main Drawer Navigation](#8-main-drawer-navigation)
- [Data Models](#data-models)
- [State Management](#state-management)
- [Dependencies](#dependencies)
- [Getting Started](#getting-started)

---

## Overview

The Meals app is a recipe browser built with Flutter. It provides 10 meal categories (Italian, Asian, German, French, etc.) and 10 sample meals. Users can filter meals by dietary preferences, tap into any meal for full recipe details, and mark meals as favorites for quick access.

---

## Features

- Browse meals organized in a 2-column category grid
- Filter meals by: Gluten-free, Lactose-free, Vegetarian, Vegan
- View full meal details: image, ingredients, and preparation steps
- Toggle favorite status on any meal with a snackbar confirmation
- Dedicated Favorites tab to revisit saved meals
- Side drawer for navigating between Meals and Filters
- Dark theme with Google Fonts (Lato) and a warm brown color scheme

---

## Project Structure

```
lib/
├── main.dart                   # App entry point, theme configuration
├── data/
│   └── dummy_data.dart         # Static list of categories and meals
├── models/
│   ├── category.dart           # Category model (id, title, color)
│   └── meal.dart               # Meal model + Complexity & Affordability enums
├── screens/
│   ├── tabs.dart               # Root screen with bottom nav + drawer + filter logic
│   ├── categories.dart         # Category grid screen
│   ├── meals.dart              # Meal list screen (used for category meals & favorites)
│   ├── meal_details.dart       # Full recipe detail screen
│   └── filters.dart            # Dietary filter settings screen
└── widgets/
    ├── category_grid_item.dart # Single tappable category card
    ├── meal_item.dart          # Meal card with image, title, and traits
    ├── meal_item_trait.dart    # Icon + label row (duration, complexity, affordability)
    ├── meal_section.dart       # Reusable section for ingredients and steps
    ├── filter_list.dart        # Reusable SwitchListTile for each filter
    └── main_drawer.dart        # Side navigation drawer
```

---

## App Workflow

### 1. App Entry Point

`main.dart` bootstraps the app by defining a dark `ThemeData` seeded from a warm brown color (`#833900`) and applying the Lato font via `google_fonts`. It renders `MyApp`, which sets `TabsScreen` as the home route.

```
main() → MyApp → TabsScreen
```

---

### 2. Tabs Screen (Home)

`TabsScreen` is the root stateful widget and acts as the central controller for the entire app. It manages:

- **Bottom navigation** between the Categories tab (index 0) and the Favorites tab (index 1)
- **Active filter state** (`Map<Filter, bool>`) initialized with all filters set to `false`
- **Favorites list** (`List<Meal>`) that tracks meals the user has starred
- **Meal filtering logic** — before passing meals down to child screens, it filters `dummyMeals` against the active filters

The `_setScreen` method handles drawer navigation. When the user selects "Filters" from the drawer, it pushes `FiltersScreen` and `await`s the returned filter map, then updates state.

---

### 3. Categories Screen

`CategoriesScreen` displays all 10 available meal categories in a `GridView` with 2 columns. Each cell is a `CategoryGridItem` — a rounded, gradient-colored card that responds to taps with an ink splash.

**User flow:**
1. User taps a category card
2. `_selectCategory` filters `availableMeals` to only those belonging to the tapped category
3. Navigates to `MealsScreen` with the filtered list and the category title as the app bar title

**Available categories:** Italian, Quick & Easy, Hamburgers, German, Light & Lovely, Exotic, Breakfast, Asian, French, Summer

---

### 4. Meals Screen

`MealsScreen` is a reusable screen used in two contexts:

- **Category meals** — receives a title and a filtered list of meals
- **Favorites** — receives no title and the global favorites list

It renders a `ListView` of `MealItem` cards. Each card shows:
- A fade-in network image (300px height)
- Meal title overlaid on a semi-transparent banner
- Three traits at the bottom: duration (minutes), complexity, and affordability

If the meals list is empty, a centered message is shown: *"Uh oh....nothing here! Try selecting a different category!"*

Tapping a meal card navigates to `MealDetailsScreen`.

---

### 5. Meal Details Screen

`MealDetailsScreen` shows the full recipe for a selected meal. It includes:

- A rounded network image (250px height) at the top
- A scrollable `ListView` body with two `MealSection` widgets:
  - **Types of Ingredients** — lists all ingredients
  - **How it is prepared** — lists all preparation steps
- A heart icon button in the app bar to toggle the meal's favorite status

Tapping the heart icon calls `onToggleFavortie`, which bubbles up to `TabsScreen` to add or remove the meal from the favorites list. A snackbar confirms the action.

---

### 6. Filters Screen

`FiltersScreen` is a stateful screen reached via the side drawer. It presents four `SwitchListTile` toggles:

| Filter        | Description                          |
|---------------|--------------------------------------|
| Gluten-free   | Only include gluten-free meals       |
| Lactose-free  | Only include lactose-free meals      |
| Vegetarian    | Only include vegetarian meals        |
| Vegan         | Only include vegan meals             |

The screen uses `PopScope` with `canPop: false` to intercept the back gesture. When the user navigates back, it pops with the current filter map as a result. `TabsScreen` receives this result and updates `_selectedFilters`, which immediately re-filters the available meals.

---

### 7. Favorites Tab

The second tab in the bottom navigation bar shows `MealsScreen` with the `_favoriteMeals` list and the title "Your Favorites". The workflow is identical to the category meals list — tapping a meal opens its detail screen where the user can also un-favorite it.

---

### 8. Main Drawer Navigation

The `MainDrawer` slides in from the left and provides two navigation options:

- **Meals** — pops the drawer and returns to the main tabs screen
- **Filters** — pops the drawer and pushes `FiltersScreen`

The drawer header displays a food icon and the app name "Cooking Up 🔥".

---

## Data Models

### `Category`
| Field   | Type     | Description                  |
|---------|----------|------------------------------|
| `id`    | `String` | Unique identifier (e.g. c1)  |
| `title` | `String` | Display name                 |
| `color` | `Color`  | Card gradient color          |

### `Meal`
| Field           | Type             | Description                          |
|-----------------|------------------|--------------------------------------|
| `id`            | `String`         | Unique identifier                    |
| `categories`    | `List<String>`   | Category IDs this meal belongs to    |
| `title`         | `String`         | Meal name                            |
| `imageUrl`      | `String`         | Remote image URL                     |
| `ingredients`   | `List<String>`   | List of ingredients                  |
| `steps`         | `List<String>`   | Preparation steps                    |
| `duration`      | `int`            | Cook time in minutes                 |
| `complexity`    | `Complexity`     | `simple`, `challenging`, or `hard`   |
| `affordability` | `Affordability`  | `affordable`, `pricey`, or `luxurious` |
| `isGlutenFree`  | `bool`           | Dietary flag                         |
| `isLactoseFree` | `bool`           | Dietary flag                         |
| `isVegan`       | `bool`           | Dietary flag                         |
| `isVegetarian`  | `bool`           | Dietary flag                         |

---

## State Management

The app uses Flutter's built-in `setState` for state management, lifted to `TabsScreen`:

- **Filter state** is owned by `TabsScreen` and passed down to `FiltersScreen` as initial values. Results are returned via `Navigator.pop()`.
- **Favorites state** is owned by `TabsScreen` and the toggle callback is passed down through `CategoriesScreen → MealsScreen → MealDetailsScreen`.
- **Filtered meals** are computed inline in `TabsScreen.build()` on every rebuild, ensuring the displayed meals always reflect the current filters.

---

## Dependencies

| Package             | Version   | Purpose                                      |
|---------------------|-----------|----------------------------------------------|
| `flutter`           | SDK       | Core framework                               |
| `google_fonts`      | ^8.0.1    | Lato font family                             |
| `transparent_image` | ^2.0.1    | Transparent placeholder for `FadeInImage`    |
| `cupertino_icons`   | ^1.0.8    | iOS-style icons                              |

---

## Getting Started

### Prerequisites
- Flutter SDK `^3.9.2`
- Dart SDK (included with Flutter)
- Android Studio / Xcode for device emulation

### Run the app

```bash
# Install dependencies
flutter pub get

# Run on a connected device or emulator
flutter run
```

### Build for release

```bash
# Android
flutter build apk

# iOS
flutter build ios
```

For more help with Flutter development, visit the [Flutter documentation](https://docs.flutter.dev/).
