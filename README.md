# React Native Learning Notes

Date: July 2, 2026

## Daily Status Summary

Today I studied React Native from a practical application development and architecture perspective, focusing on:

- Project structure and maintainable folder organization
- Reusable component design and screen composition
- Navigation flow and route management
- API integration patterns with loading, error, and success states
- State management approach for screen-level and shared data
- Debugging workflow using Metro, emulator logs, and Android tooling
- Android build and run flow for local validation

This helped me understand how to structure a React Native application in a scalable and maintainable way for real project development.

## 1. What React Native Is

React Native is a framework for building mobile applications using React concepts and JavaScript or TypeScript.

In React for web, UI is rendered using browser elements. In React Native, UI is built using React Native components such as `View`, `Text`, `Image`, `ScrollView`, `FlatList`, and `TextInput`.

The development model is still React-based:

- screens are composed from components
- data is passed through props
- UI changes are controlled through state
- side effects are handled through hooks
- styling is defined in JavaScript using `StyleSheet`

Simple screen example:

```tsx
import React from "react";
import { SafeAreaView, StyleSheet, Text, View } from "react-native";

export function HomeScreen() {
  return (
    <SafeAreaView style={styles.page}>
      <View style={styles.card}>
        <Text style={styles.title}>Dashboard</Text>
        <Text style={styles.subtitle}>Mobile app screen built with React Native</Text>
      </View>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  page: {
    flex: 1,
    backgroundColor: "#f5f7fb",
    padding: 16,
  },
  card: {
    backgroundColor: "#ffffff",
    borderRadius: 12,
    padding: 16,
  },
  title: {
    fontSize: 22,
    fontWeight: "700",
    color: "#111827",
  },
  subtitle: {
    marginTop: 6,
    fontSize: 14,
    color: "#6b7280",
  },
});
```

## 2. Project Structure

A clean folder structure is important because React Native applications usually grow into multiple screens, shared components, API services, hooks, constants, and utilities.

Recommended structure:

```text
src/
  app/
    App.tsx
    navigation/
      RootNavigator.tsx
      routes.ts
  screens/
    HomeScreen.tsx
    DetailsScreen.tsx
    SettingsScreen.tsx
  components/
    AppButton.tsx
    MetricCard.tsx
    EmptyState.tsx
    LoadingState.tsx
  services/
    apiClient.ts
    dashboardService.ts
  hooks/
    useDashboardData.ts
  state/
    AppContext.tsx
  constants/
    colors.ts
    spacing.ts
  utils/
    formatters.ts
```

### Why This Structure Helps

| Folder | Purpose |
| --- | --- |
| `screens` | Owns full screen-level UI and screen flow. |
| `components` | Stores reusable UI pieces used across screens. |
| `services` | Keeps API and data access code separate from UI. |
| `hooks` | Stores reusable stateful logic. |
| `state` | Stores shared app-level state when needed. |
| `constants` | Keeps design tokens and static values consistent. |
| `utils` | Stores pure helper functions. |

Good rule:

> Screens should compose behavior. Components should render reusable UI. Services should handle external data.

## 3. Reusable Component Design

Reusable components prevent duplication and make screens easier to maintain.

Instead of repeating the same card layout in multiple screens, create a reusable component.

Example:

```tsx
import React from "react";
import { StyleSheet, Text, View } from "react-native";

type MetricCardProps = {
  label: string;
  value: string;
  status?: "good" | "warning" | "danger";
};

export function MetricCard({ label, value, status = "good" }: MetricCardProps) {
  return (
    <View style={styles.card}>
      <Text style={styles.label}>{label}</Text>
      <Text style={[styles.value, styles[status]]}>{value}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: "#ffffff",
    borderRadius: 10,
    padding: 14,
    borderWidth: 1,
    borderColor: "#e5e7eb",
  },
  label: {
    fontSize: 13,
    color: "#6b7280",
  },
  value: {
    marginTop: 8,
    fontSize: 24,
    fontWeight: "700",
  },
  good: {
    color: "#047857",
  },
  warning: {
    color: "#b45309",
  },
  danger: {
    color: "#b91c1c",
  },
});
```

Usage:

```tsx
<MetricCard label="Success Rate" value="96%" status="good" />
<MetricCard label="Open Issues" value="12" status="warning" />
```

### Component Design Guidelines

- Keep components focused on one responsibility.
- Pass data through props.
- Avoid putting API calls inside reusable UI components.
- Keep layout components reusable and predictable.
- Use TypeScript props to make usage clear.
- Keep screen-specific logic in screens or hooks, not inside generic components.

## 4. Screen Composition

A screen should combine smaller components and control the flow for that page.

Example:

```tsx
import React from "react";
import { ScrollView, StyleSheet, Text, View } from "react-native";
import { MetricCard } from "../components/MetricCard";

export function DashboardScreen() {
  return (
    <ScrollView contentContainerStyle={styles.container}>
      <Text style={styles.heading}>Dashboard Overview</Text>

      <View style={styles.grid}>
        <MetricCard label="Revenue" value="$42K" />
        <MetricCard label="Delivery" value="94%" />
        <MetricCard label="Risk Items" value="7" status="warning" />
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    gap: 16,
  },
  heading: {
    fontSize: 24,
    fontWeight: "700",
  },
  grid: {
    gap: 12,
  },
});
```

This keeps the screen readable because the repeated UI is handled by `MetricCard`.

## 5. Navigation Flow

Most real React Native apps have multiple screens.

Common navigation patterns:

- stack navigation: screen-to-screen flow
- tab navigation: bottom tabs or top tabs
- nested navigation: tabs inside stack or stack inside tabs
- route params: pass small data to the next screen

Example route names:

```ts
export type RootStackParamList = {
  Home: undefined;
  Details: { itemId: string };
  Settings: undefined;
};
```

Example navigation:

```tsx
navigation.navigate("Details", { itemId: "1001" });
```

Example receiving params:

```tsx
export function DetailsScreen({ route }) {
  const { itemId } = route.params;

  return <Text>Selected item: {itemId}</Text>;
}
```

### Navigation Best Practices

- Keep route names centralized.
- Pass only required data through route params.
- Do not pass large objects between screens.
- Fetch detail data inside the detail screen using an ID.
- Keep navigation setup separate from screen UI.

Good approach:

```text
User taps list item
  -> navigate to Details with itemId
  -> Details screen loads detail using itemId
  -> screen shows loading, success, or error state
```

## 6. API Integration

API calls should not be scattered directly inside UI components. A service layer keeps API logic maintainable.

Recommended flow:

```text
Screen
  -> custom hook
  -> service function
  -> API client
  -> backend
```

Example API client:

```ts
const API_BASE_URL = "https://example-api.com";

export async function apiGet<T>(path: string): Promise<T> {
  const response = await fetch(`${API_BASE_URL}${path}`);

  if (!response.ok) {
    throw new Error(`Request failed with status ${response.status}`);
  }

  return response.json() as Promise<T>;
}
```

Example service:

```ts
import { apiGet } from "./apiClient";

export type DashboardSummary = {
  totalItems: number;
  successRate: number;
  openIssues: number;
};

export function fetchDashboardSummary() {
  return apiGet<DashboardSummary>("/dashboard/summary");
}
```

Example hook:

```tsx
import { useEffect, useState } from "react";
import { fetchDashboardSummary, type DashboardSummary } from "../services/dashboardService";

export function useDashboardSummary() {
  const [data, setData] = useState<DashboardSummary | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let isMounted = true;

    async function loadData() {
      try {
        setIsLoading(true);
        setError(null);
        const result = await fetchDashboardSummary();
        if (isMounted) {
          setData(result);
        }
      } catch (caughtError) {
        if (isMounted) {
          setError(caughtError instanceof Error ? caughtError.message : "Unable to load data");
        }
      } finally {
        if (isMounted) {
          setIsLoading(false);
        }
      }
    }

    loadData();

    return () => {
      isMounted = false;
    };
  }, []);

  return { data, isLoading, error };
}
```

Example screen usage:

```tsx
export function DashboardScreen() {
  const { data, isLoading, error } = useDashboardSummary();

  if (isLoading) {
    return <LoadingState message="Loading dashboard..." />;
  }

  if (error) {
    return <ErrorState message={error} />;
  }

  if (!data) {
    return <EmptyState message="No dashboard data available." />;
  }

  return (
    <>
      <MetricCard label="Total Items" value={String(data.totalItems)} />
      <MetricCard label="Success Rate" value={`${data.successRate}%`} />
      <MetricCard label="Open Issues" value={String(data.openIssues)} />
    </>
  );
}
```

### API Handling Best Practices

- Keep fetch logic outside UI components.
- Always handle loading, error, empty, and success states.
- Keep backend response types explicit.
- Avoid duplicate API logic across screens.
- Add retry support for important flows.
- Use request cancellation or mounted checks to avoid updating unmounted screens.

## 7. State Management

React Native state can be handled at different levels.

| State Type | Best Place |
| --- | --- |
| Button loading state | Local screen/component state |
| Form field values | Local state or form library |
| Data loaded for one screen | Custom hook |
| User/session/theme | Context or global state |
| Large shared app state | State library when needed |

### Local State Example

```tsx
const [searchText, setSearchText] = useState("");
```

Use local state when only one component or screen needs the data.

### Shared Hook Example

Use a custom hook when multiple screens need the same loading logic.

```tsx
export function useSearchQuery() {
  const [query, setQuery] = useState("");

  function clearQuery() {
    setQuery("");
  }

  return { query, setQuery, clearQuery };
}
```

### Context Example

Use context for cross-screen values such as theme or logged-in user details.

```tsx
import React, { createContext, useContext, useState } from "react";

type ThemeMode = "light" | "dark";

const ThemeContext = createContext<{
  mode: ThemeMode;
  setMode: (mode: ThemeMode) => void;
} | null>(null);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [mode, setMode] = useState<ThemeMode>("light");

  return <ThemeContext.Provider value={{ mode, setMode }}>{children}</ThemeContext.Provider>;
}

export function useThemeMode() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useThemeMode must be used inside ThemeProvider");
  }
  return context;
}
```

### State Management Guidelines

- Start with local state.
- Move logic into custom hooks when repeated.
- Use context for small shared app-level state.
- Add a state library only when the application clearly needs it.
- Avoid global state for everything.
- Keep server data and UI state conceptually separate.

## 8. Loading, Error, Empty, And Success States

Production screens should not only handle the happy path.

Every API-driven screen should define:

- loading state
- error state
- empty state
- success state

Example:

```tsx
if (isLoading) {
  return <LoadingState message="Loading records..." />;
}

if (error) {
  return <ErrorState message={error} onRetry={reload} />;
}

if (records.length === 0) {
  return <EmptyState message="No records found." />;
}

return <RecordList records={records} />;
```

Why this matters:

- users understand what is happening
- support issues are easier to debug
- screens behave predictably during slow network conditions
- error recovery becomes part of the design

## 9. Styling And Layout

React Native uses JavaScript-based styling.

Common layout concepts:

- `flex: 1` fills available space
- `flexDirection` controls row or column layout
- `gap`, `padding`, and `margin` control spacing
- `alignItems` controls cross-axis alignment
- `justifyContent` controls main-axis alignment

Example:

```tsx
const styles = StyleSheet.create({
  row: {
    flexDirection: "row",
    alignItems: "center",
    justifyContent: "space-between",
    gap: 12,
  },
  page: {
    flex: 1,
    padding: 16,
    backgroundColor: "#f9fafb",
  },
});
```

### Styling Best Practices

- Define shared colors and spacing constants.
- Keep screen-level styles close to the screen.
- Move repeated UI styling into components.
- Avoid hardcoded repeated values across many files.
- Test layouts on different screen sizes.
- Ensure touch targets are large enough for mobile usage.

Example design tokens:

```ts
export const colors = {
  background: "#f9fafb",
  surface: "#ffffff",
  text: "#111827",
  mutedText: "#6b7280",
  primary: "#2563eb",
};

export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
};
```

## 10. Lists And Performance

For lists, `FlatList` is preferred over manually mapping a large array inside `ScrollView`.

Example:

```tsx
import { FlatList } from "react-native";

export function RecordList({ records }) {
  return (
    <FlatList
      data={records}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <RecordRow record={item} />}
    />
  );
}
```

Performance practices:

- Use stable `keyExtractor`.
- Keep list rows lightweight.
- Memoize expensive row components when needed.
- Avoid inline heavy calculations inside `renderItem`.
- Use pagination for large data.
- Use thumbnails instead of heavy images in list rows.
- Profile before optimizing too early.

Example with memoized row:

```tsx
import React from "react";

const RecordRow = React.memo(function RecordRow({ record }) {
  return (
    <View>
      <Text>{record.title}</Text>
      <Text>{record.status}</Text>
    </View>
  );
});
```

## 11. Debugging Workflow

React Native debugging should be done across multiple layers of the development setup.

| Tool | Use |
| --- | --- |
| Metro terminal | JavaScript logs, bundle errors, reload status |
| React Native DevTools | component inspection and debugging |
| Android Studio Logcat | Android runtime logs |
| Emulator | device behavior, layout, keyboard, permissions |
| Network inspector/proxy | API request and response debugging |

Useful logging pattern:

```tsx
console.log("[DashboardScreen] Loading dashboard summary");
console.log("[DashboardScreen] API response", response);
console.error("[DashboardScreen] API failed", error);
```

Debugging checklist:

- Confirm Metro is running.
- Confirm emulator/device is connected.
- Check if the app is installed successfully.
- Check Metro logs for JavaScript errors.
- Check Logcat for Android-side runtime errors.
- Confirm API base URL and network connectivity.
- Reproduce the issue with exact steps.
- Add targeted logs near the suspected failure point.

## 12. Android Build And Run Flow

React Native Android development usually involves:

```text
Install JS dependencies
  -> start Metro
  -> build Android app with Gradle
  -> install app on emulator/device
  -> app loads JS bundle from Metro during development
```

Common commands:

```bash
npm install
npm run android
```

Or with React Native CLI:

```bash
npx react-native start
npx react-native run-android
```

Typical Android output locations:

```text
android/app/build/outputs/apk/debug/
android/app/build/outputs/apk/release/
```

### Android Build Concepts

| Concept | Meaning |
| --- | --- |
| Metro | Development server that serves the JavaScript bundle. |
| Gradle | Android build system used to compile and package the app. |
| Debug APK | APK used during development and testing. |
| Release APK/AAB | Production build artifact for distribution. |
| Emulator | Local Android device simulation for development testing. |
| Logcat | Android logging tool used for runtime investigation. |

### Common Build Issues To Check

- wrong Node version
- missing Android SDK
- wrong Java/JDK version
- emulator not started
- Gradle cache issue
- package installation failure
- Metro not running
- port conflict

## 13. Practical Screen Development Flow

Recommended flow for building a new screen:

```text
1. Define screen responsibility.
2. Identify data needed by the screen.
3. Create or reuse API service.
4. Create a custom hook for loading state.
5. Build reusable UI components.
6. Compose the screen.
7. Add loading, error, empty, and success states.
8. Add navigation route and params.
9. Add logs for important user actions.
10. Test on emulator with different data states.
```

Example:

```text
Requirement:
Show a list of tasks and open details on tap.

Implementation:
- TaskListScreen
- TaskDetailsScreen
- TaskCard component
- taskService.ts
- useTasks hook
- navigation route: TaskDetails with taskId
```

## 14. Suggested Learning Roadmap

### Day 1: App Structure And UI Composition

- Understand project folders.
- Build screens using components.
- Use `StyleSheet` and Flexbox.
- Create reusable cards, buttons, and layout components.

### Day 2: Navigation And API Handling

- Add stack navigation.
- Add tab navigation if needed.
- Pass route params.
- Build API service layer.
- Handle loading, error, empty, and success states.

### Day 3: State And Forms

- Use local state for screen behavior.
- Use custom hooks for repeated logic.
- Use context for shared state.
- Build controlled forms.
- Add validation and submit states.

### Day 4: Debugging, Build Flow, And Performance

- Run app on Android emulator.
- Review Metro logs and Logcat.
- Build debug APK.
- Optimize list rendering.
- Review re-render causes.
- Prepare project structure recommendation.

## 15. Manager-Ready Summary

Today I studied React Native from a practical application development and architecture perspective. The focus was on project structure, reusable components, navigation, API integration, state management, debugging workflow, performance considerations, and Android build flow. The key outcome is a clearer understanding of how to structure React Native applications in a scalable and maintainable way for real project development.

## Reference Links

- React Native documentation: https://reactnative.dev/docs/getting-started
- React Native environment setup: https://reactnative.dev/docs/environment-setup
- React Native navigation overview: https://reactnative.dev/docs/navigation
- React Navigation documentation: https://reactnavigation.org/docs/getting-started/
- React Native performance overview: https://reactnative.dev/docs/performance
- React Native FlatList optimization: https://reactnative.dev/docs/optimizing-flatlist-configuration
