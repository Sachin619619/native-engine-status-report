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
// Import React so this file can define a React component.
import React from "react";
// Import the React Native building blocks used by this screen.
import { SafeAreaView, StyleSheet, Text, View } from "react-native";

// Export a screen component so it can be used by navigation or the app entry point.
export function HomeScreen() {
  // Return the UI tree rendered by this screen with SafeAreaView as the root.
  return (
    <SafeAreaView style={styles.page}>
      {/* View creates a container for the card content. */}
      <View style={styles.card}>
        {/* Text renders the main heading on the screen. */}
        <Text style={styles.title}>Dashboard</Text>
        {/* Text renders secondary supporting content below the heading. */}
        <Text style={styles.subtitle}>Mobile app screen built with React Native</Text>
      {/* Close the card container. */}
      </View>
    {/* Close the safe area container. */}
    </SafeAreaView>
  );
}

// Create a centralized style object for this screen.
const styles = StyleSheet.create({
  // Style for the full page container.
  page: {
    // Make the page fill all available screen height.
    flex: 1,
    // Apply a light background color to the full screen.
    backgroundColor: "#f5f7fb",
    // Add spacing around the screen content.
    padding: 16,
  },
  // Style for the white dashboard card.
  card: {
    // Set the card background color.
    backgroundColor: "#ffffff",
    // Round the card corners.
    borderRadius: 12,
    // Add inner spacing inside the card.
    padding: 16,
  },
  // Style for the main title text.
  title: {
    // Set title text size.
    fontSize: 22,
    // Make the title bold.
    fontWeight: "700",
    // Set title color.
    color: "#111827",
  },
  // Style for the subtitle text.
  subtitle: {
    // Add space between title and subtitle.
    marginTop: 6,
    // Set subtitle text size.
    fontSize: 14,
    // Set subtitle color.
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
// Import React to define a reusable component.
import React from "react";
// Import React Native primitives used by the card.
import { StyleSheet, Text, View } from "react-native";

// Define the props accepted by the MetricCard component.
type MetricCardProps = {
  // Label describes what metric is being shown.
  label: string;
  // Value is the formatted value displayed in the card.
  value: string;
  // Status controls the visual color of the value.
  status?: "good" | "warning" | "danger";
};

// Export a reusable card component for metric-style values.
export function MetricCard({ label, value, status = "good" }: MetricCardProps) {
  // Return the visual structure of the metric card with View as the root.
  return (
    <View style={styles.card}>
      {/* Render the metric label. */}
      <Text style={styles.label}>{label}</Text>
      {/* Render the metric value and apply status-specific color. */}
      <Text style={[styles.value, styles[status]]}>{value}</Text>
    {/* Close the card container. */}
    </View>
  );
}

// Create local styles for the MetricCard component.
const styles = StyleSheet.create({
  // Card container style.
  card: {
    // Set card background.
    backgroundColor: "#ffffff",
    // Round card corners.
    borderRadius: 10,
    // Add inner spacing.
    padding: 14,
    // Add a thin border.
    borderWidth: 1,
    // Set border color.
    borderColor: "#e5e7eb",
  },
  // Label text style.
  label: {
    // Set label font size.
    fontSize: 13,
    // Use muted color for label text.
    color: "#6b7280",
  },
  // Value text style.
  value: {
    // Add space above the value.
    marginTop: 8,
    // Make the value visually prominent.
    fontSize: 24,
    // Use bold weight for the value.
    fontWeight: "700",
  },
  // Color used when the metric is healthy.
  good: {
    // Green value color.
    color: "#047857",
  },
  // Color used when the metric needs attention.
  warning: {
    // Amber value color.
    color: "#b45309",
  },
  // Color used when the metric is risky.
  danger: {
    // Red value color.
    color: "#b91c1c",
  },
});
```

Usage:

```tsx
{/* Render a healthy success-rate metric card. */}
<MetricCard label="Success Rate" value="96%" status="good" />
{/* Render a warning-state open-issues metric card. */}
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
// Import React to create the screen component.
import React from "react";
// Import React Native primitives used to render the screen.
import { ScrollView, StyleSheet, Text, View } from "react-native";
// Import the reusable MetricCard component.
import { MetricCard } from "../components/MetricCard";

// Export the dashboard screen.
export function DashboardScreen() {
  // Return scrollable screen content with ScrollView as the root.
  return (
    <ScrollView contentContainerStyle={styles.container}>
      {/* Render the screen heading. */}
      <Text style={styles.heading}>Dashboard Overview</Text>

      {/* Group metric cards inside a vertical grid container. */}
      <View style={styles.grid}>
        {/* Render a revenue metric card. */}
        <MetricCard label="Revenue" value="$42K" />
        {/* Render a delivery metric card. */}
        <MetricCard label="Delivery" value="94%" />
        {/* Render a warning metric card for risk items. */}
        <MetricCard label="Risk Items" value="7" status="warning" />
      {/* Close the grid container. */}
      </View>
    {/* Close the scrollable page. */}
    </ScrollView>
  );
}

// Create local styles for this dashboard screen.
const styles = StyleSheet.create({
  // Style applied to ScrollView content.
  container: {
    // Add screen padding.
    padding: 16,
    // Add spacing between direct child elements.
    gap: 16,
  },
  // Style for the page heading.
  heading: {
    // Set heading size.
    fontSize: 24,
    // Make heading bold.
    fontWeight: "700",
  },
  // Style for the metric-card group.
  grid: {
    // Add spacing between metric cards.
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
// Define all stack route names and the params each route accepts.
export type RootStackParamList = {
  // Home route does not require route params.
  Home: undefined;
  // Details route requires an itemId string.
  Details: { itemId: string };
  // Settings route does not require route params.
  Settings: undefined;
};
```

Example navigation:

```tsx
// Navigate to the Details screen and pass the selected item ID.
navigation.navigate("Details", { itemId: "1001" });
```

Example receiving params:

```tsx
// Define the Details screen and receive navigation route props.
export function DetailsScreen({ route }) {
  // Read itemId from route params.
  const { itemId } = route.params;

  // Render the selected item ID on the screen.
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
// Store the backend base URL in one place.
const API_BASE_URL = "https://example-api.com";

// Create a reusable GET helper with a generic response type.
export async function apiGet<T>(path: string): Promise<T> {
  // Make the network request by joining base URL and path.
  const response = await fetch(`${API_BASE_URL}${path}`);

  // Treat non-2xx responses as errors.
  if (!response.ok) {
    // Throw an error with the HTTP status for easier debugging.
    throw new Error(`Request failed with status ${response.status}`);
  }

  // Parse and return the JSON response as the expected type.
  return response.json() as Promise<T>;
}
```

Example service:

```ts
// Import the shared API GET helper.
import { apiGet } from "./apiClient";

// Define the expected dashboard summary response shape.
export type DashboardSummary = {
  // Total number of items shown on the dashboard.
  totalItems: number;
  // Success percentage returned by the backend.
  successRate: number;
  // Number of open issues returned by the backend.
  openIssues: number;
};

// Create a service function for loading dashboard summary data.
export function fetchDashboardSummary() {
  // Call the dashboard summary endpoint and type the response.
  return apiGet<DashboardSummary>("/dashboard/summary");
}
```

Example hook:

```tsx
// Import React hooks needed for lifecycle and state.
import { useEffect, useState } from "react";
// Import the service function and response type.
import { fetchDashboardSummary, type DashboardSummary } from "../services/dashboardService";

// Create a custom hook that owns dashboard loading logic.
export function useDashboardSummary() {
  // Store the loaded dashboard data.
  const [data, setData] = useState<DashboardSummary | null>(null);
  // Store whether the request is currently loading.
  const [isLoading, setIsLoading] = useState(true);
  // Store an error message when the request fails.
  const [error, setError] = useState<string | null>(null);

  // Run the data load when the hook is first used.
  useEffect(() => {
    // Track whether the component is still mounted.
    let isMounted = true;

    // Define the async data-loading function.
    async function loadData() {
      // Start the protected request block.
      try {
        // Mark the screen as loading.
        setIsLoading(true);
        // Clear any old error before starting a new request.
        setError(null);
        // Call the service layer to fetch dashboard data.
        const result = await fetchDashboardSummary();
        // Only update state if the component is still mounted.
        if (isMounted) {
          // Store the successful API result.
          setData(result);
        }
      // Handle request or parsing errors.
      } catch (caughtError) {
        // Only update state if the component is still mounted.
        if (isMounted) {
          // Store a readable error message.
          setError(caughtError instanceof Error ? caughtError.message : "Unable to load data");
        }
      // Always run cleanup for the loading flag.
      } finally {
        // Only update state if the component is still mounted.
        if (isMounted) {
          // Stop the loading state.
          setIsLoading(false);
        }
      }
    }

    // Trigger the data load.
    loadData();

    // Cleanup runs when the component using this hook unmounts.
    return () => {
      // Prevent state updates after unmount.
      isMounted = false;
    };
  // Empty dependency array means this effect runs once on mount.
  }, []);

  // Return data and request state to the screen.
  return { data, isLoading, error };
}
```

Example screen usage:

```tsx
// Define the dashboard screen component.
export function DashboardScreen() {
  // Use the custom hook to load dashboard data and request status.
  const { data, isLoading, error } = useDashboardSummary();

  // Show loading UI while the request is in progress.
  if (isLoading) {
    // Return a reusable loading component.
    return <LoadingState message="Loading dashboard..." />;
  }

  // Show error UI if the request failed.
  if (error) {
    // Return a reusable error component with the error message.
    return <ErrorState message={error} />;
  }

  // Show empty UI if the request succeeded but no data is available.
  if (!data) {
    // Return a reusable empty-state component.
    return <EmptyState message="No dashboard data available." />;
  }

  // Render the successful dashboard state using a Fragment root.
  return (
    <>
      {/* Show total item count. */}
      <MetricCard label="Total Items" value={String(data.totalItems)} />
      {/* Show success-rate percentage. */}
      <MetricCard label="Success Rate" value={`${data.successRate}%`} />
      {/* Show open issue count. */}
      <MetricCard label="Open Issues" value={String(data.openIssues)} />
    {/* Close the fragment. */}
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
// Create local state for the current search text.
const [searchText, setSearchText] = useState("");
```

Use local state when only one component or screen needs the data.

### Shared Hook Example

Use a custom hook when multiple screens need the same loading logic.

```tsx
// Export a custom hook that owns search-query behavior.
export function useSearchQuery() {
  // Store the current search query.
  const [query, setQuery] = useState("");

  // Define a helper function to reset the query.
  function clearQuery() {
    // Set query back to an empty string.
    setQuery("");
  }

  // Expose the query value, setter, and clear helper.
  return { query, setQuery, clearQuery };
}
```

### Context Example

Use context for cross-screen values such as theme or logged-in user details.

```tsx
// Import React and hooks needed to create context.
import React, { createContext, useContext, useState } from "react";

// Define the allowed theme modes.
type ThemeMode = "light" | "dark";

// Create a context for theme state and updater function.
const ThemeContext = createContext<{
  // Current theme mode.
  mode: ThemeMode;
  // Function used to update the theme mode.
  setMode: (mode: ThemeMode) => void;
// Default value is null so usage outside the provider can be detected.
} | null>(null);

// Create a provider component that wraps screens needing theme access.
export function ThemeProvider({ children }: { children: React.ReactNode }) {
  // Store the current theme mode.
  const [mode, setMode] = useState<ThemeMode>("light");

  // Provide mode and setMode to all children.
  return <ThemeContext.Provider value={{ mode, setMode }}>{children}</ThemeContext.Provider>;
}

// Create a custom hook for consuming theme context.
export function useThemeMode() {
  // Read the current context value.
  const context = useContext(ThemeContext);
  // Guard against using the hook outside ThemeProvider.
  if (!context) {
    // Throw a clear developer error.
    throw new Error("useThemeMode must be used inside ThemeProvider");
  }
  // Return the validated context.
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
// Check whether the request is currently loading.
if (isLoading) {
  // Render loading UI while data is being fetched.
  return <LoadingState message="Loading records..." />;
}

// Check whether the request failed.
if (error) {
  // Render error UI and provide retry support.
  return <ErrorState message={error} onRetry={reload} />;
}

// Check whether the request succeeded with no records.
if (records.length === 0) {
  // Render empty-state UI when there is no data to show.
  return <EmptyState message="No records found." />;
}

// Render the normal success state when records are available.
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
// Create styles for reusable layout patterns.
const styles = StyleSheet.create({
  // Row layout style for horizontally aligned content.
  row: {
    // Arrange children from left to right.
    flexDirection: "row",
    // Align children vertically in the center.
    alignItems: "center",
    // Push first and last child to opposite ends.
    justifyContent: "space-between",
    // Add spacing between children.
    gap: 12,
  },
  // Page container style.
  page: {
    // Fill the available screen area.
    flex: 1,
    // Add page padding.
    padding: 16,
    // Set a light page background.
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
// Define shared colors used across the app.
export const colors = {
  // App background color.
  background: "#f9fafb",
  // Card or surface background color.
  surface: "#ffffff",
  // Main text color.
  text: "#111827",
  // Secondary text color.
  mutedText: "#6b7280",
  // Primary action color.
  primary: "#2563eb",
};

// Define shared spacing values used across the app.
export const spacing = {
  // Extra-small spacing.
  xs: 4,
  // Small spacing.
  sm: 8,
  // Medium spacing.
  md: 16,
  // Large spacing.
  lg: 24,
  // Extra-large spacing.
  xl: 32,
};
```

## 10. Lists And Performance

For lists, `FlatList` is preferred over manually mapping a large array inside `ScrollView`.

Example:

```tsx
// Import FlatList for efficient list rendering.
import { FlatList } from "react-native";

// Define a reusable list component for records.
export function RecordList({ records }) {
  // Return a virtualized FlatList for better list performance.
  return (
    <FlatList
      // Provide the array of records to render.
      data={records}
      // Provide a stable unique key for each row.
      keyExtractor={(item) => item.id}
      // Render each row using the RecordRow component.
      renderItem={({ item }) => <RecordRow record={item} />}
    // Close the FlatList component.
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
// Import React so React.memo can be used.
import React from "react";

// Memoize the row so it avoids unnecessary re-renders when props do not change.
const RecordRow = React.memo(function RecordRow({ record }) {
  // Return row UI for a single record with View as the root.
  return (
    <View>
      {/* Render the record title. */}
      <Text>{record.title}</Text>
      {/* Render the record status. */}
      <Text>{record.status}</Text>
    {/* Close the row container. */}
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
// Log when dashboard loading starts.
console.log("[DashboardScreen] Loading dashboard summary");
// Log the successful API response for debugging.
console.log("[DashboardScreen] API response", response);
// Log the API failure with error details.
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
# Install JavaScript dependencies from package.json.
npm install
# Build and run the Android app on a connected emulator or device.
npm run android
```

Or with React Native CLI:

```bash
# Start the Metro development server.
npx react-native start
# Build, install, and launch the Android app.
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
