# TrackMyJob Design System

This comprehensive guide outlines the design system for TrackMyJob, covering both design principles and implementation details for web and mobile platforms.

## Table of Contents

- [Design Tokens](#design-tokens)
  - [Colors](#colors)
  - [Typography](#typography)
  - [Spacing](#spacing)
  - [Elevation & Shadows](#elevation--shadows)
- [Theme System](#theme-system)
  - [Web Implementation](#web-implementation)
  - [Mobile Implementation](#mobile-implementation)
- [Components](#components)
  - [Web Components](#web-components)
  - [Mobile Components](#mobile-components)
- [Best Practices](#best-practices)
- [Common Patterns](#common-patterns)

## Design Tokens

### Colors

Our color system is based on a carefully selected palette that ensures consistency across platforms while maintaining accessibility standards.

```typescript
// shared/design-system/colors.ts
export const colors = {
  primary: {
    main: "#2563eb", // blue-600
    light: "#60a5fa", // blue-400
    dark: "#1d4ed8", // blue-700
    contrast: "#ffffff",
  },
  secondary: {
    main: "#4f46e5", // indigo-600
    light: "#818cf8", // indigo-400
    dark: "#4338ca", // indigo-700
    contrast: "#ffffff",
  },
  background: {
    default: "#ffffff",
    paper: "#f3f4f6", // gray-100
  },
  text: {
    primary: "#111827", // gray-900
    secondary: "#4b5563", // gray-600
    disabled: "#9ca3af", // gray-400
  },
  error: {
    main: "#dc2626", // red-600
    light: "#f87171", // red-400
    dark: "#b91c1c", // red-700
  },
  success: {
    main: "#16a34a", // green-600
    light: "#4ade80", // green-400
    dark: "#15803d", // green-700
  },
  warning: {
    main: "#ca8a04", // yellow-600
    light: "#facc15", // yellow-400
    dark: "#a16207", // yellow-700
  },
};
```

### Typography

Our typography system is designed to maintain readability and hierarchy across different platforms while adapting to platform-specific best practices.

```typescript
// shared/design-system/typography.ts
export const typography = {
  fontConfig: {
    regular: {
      fontFamily: "System",
      fontWeight: "normal",
    },
    medium: {
      fontFamily: "System",
      fontWeight: "500",
    },
    bold: {
      fontFamily: "System",
      fontWeight: "bold",
    },
  },
  variants: {
    displayLarge: {
      fontSize: 57,
      letterSpacing: 0,
      lineHeight: 64,
      fontWeight: "bold",
    },
    displayMedium: {
      fontSize: 45,
      letterSpacing: 0,
      lineHeight: 52,
      fontWeight: "bold",
    },
    displaySmall: {
      fontSize: 36,
      letterSpacing: 0,
      lineHeight: 44,
      fontWeight: "bold",
    },
    headlineLarge: {
      fontSize: 32,
      letterSpacing: 0,
      lineHeight: 40,
      fontWeight: "bold",
    },
    headlineMedium: {
      fontSize: 28,
      letterSpacing: 0,
      lineHeight: 36,
      fontWeight: "bold",
    },
    headlineSmall: {
      fontSize: 24,
      letterSpacing: 0,
      lineHeight: 32,
      fontWeight: "bold",
    },
    titleLarge: {
      fontSize: 22,
      letterSpacing: 0,
      lineHeight: 28,
      fontWeight: "500",
    },
    titleMedium: {
      fontSize: 16,
      letterSpacing: 0.15,
      lineHeight: 24,
      fontWeight: "500",
    },
    titleSmall: {
      fontSize: 14,
      letterSpacing: 0.1,
      lineHeight: 20,
      fontWeight: "500",
    },
    bodyLarge: {
      fontSize: 16,
      letterSpacing: 0.15,
      lineHeight: 24,
    },
    bodyMedium: {
      fontSize: 14,
      letterSpacing: 0.25,
      lineHeight: 20,
    },
    bodySmall: {
      fontSize: 12,
      letterSpacing: 0.4,
      lineHeight: 16,
    },
    labelLarge: {
      fontSize: 14,
      letterSpacing: 0.1,
      lineHeight: 20,
      fontWeight: "500",
    },
    labelMedium: {
      fontSize: 12,
      letterSpacing: 0.5,
      lineHeight: 16,
      fontWeight: "500",
    },
    labelSmall: {
      fontSize: 11,
      letterSpacing: 0.5,
      lineHeight: 16,
      fontWeight: "500",
    },
  },
};
```

### Spacing

Our spacing system uses a consistent scale across platforms to maintain visual rhythm and component alignment.

```typescript
// shared/design-system/spacing.ts
export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  xxl: 48,
};
```

## Theme System

### Web Implementation

For web, we use Tailwind CSS with a custom configuration that matches our design tokens:

```typescript
// apps/web/tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: colors.primary,
        secondary: colors.secondary,
        // ... other colors
      },
      spacing: spacing,
      typography: (theme) => ({
        DEFAULT: {
          css: {
            color: theme("colors.gray.900"),
            // ... typography styles
          },
        },
      }),
    },
  },
};
```

### Mobile Implementation

For mobile, we use React Native Paper's theming system:

```typescript
// apps/mobile/src/theme/index.ts
import { MD3LightTheme, configureFonts } from "react-native-paper";
import { colors } from "./colors";
import { typography } from "./typography";
import { spacing } from "./spacing";

export const theme = {
  ...MD3LightTheme,
  colors: {
    ...MD3LightTheme.colors,
    ...colors,
  },
  fonts: configureFonts({
    config: typography.fontConfig,
  }),
  spacing,
  roundness: 8,
};

// Usage in App.tsx
import { PaperProvider } from "react-native-paper";
import { theme } from "./theme";

export default function App() {
  return <PaperProvider theme={theme}>{/* Your app content */}</PaperProvider>;
}
```

## Components

### Web Components

Example of a Button component for web:

```typescript
// apps/web/components/Button.tsx
import { cva } from "class-variance-authority";

const buttonStyles = cva(
  "rounded-lg font-medium focus:outline-none focus:ring-2 focus:ring-offset-2",
  {
    variants: {
      variant: {
        primary: "bg-primary-main text-white hover:bg-primary-dark",
        secondary: "bg-secondary-main text-white hover:bg-secondary-dark",
        outline:
          "border-2 border-primary-main text-primary-main hover:bg-primary-50",
      },
      size: {
        small: "px-3 py-1.5 text-sm",
        medium: "px-4 py-2 text-base",
        large: "px-6 py-3 text-lg",
      },
    },
    defaultVariants: {
      variant: "primary",
      size: "medium",
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary" | "outline";
  size?: "small" | "medium" | "large";
}

export function Button({
  variant,
  size,
  className,
  children,
  ...props
}: ButtonProps) {
  return (
    <button className={buttonStyles({ variant, size, className })} {...props}>
      {children}
    </button>
  );
}
```

### Mobile Components

Example of a Button component for mobile:

```typescript
// apps/mobile/src/components/Button/index.tsx
import { StyleSheet } from "react-native";
import { Button as PaperButton, useTheme } from "react-native-paper";

type ButtonVariant = "primary" | "secondary" | "outline" | "text";

interface ButtonProps {
  variant?: ButtonVariant;
  size?: "small" | "medium" | "large";
  onPress: () => void;
  disabled?: boolean;
  loading?: boolean;
  icon?: string;
  children: React.ReactNode;
}

export function Button({
  variant = "primary",
  size = "medium",
  onPress,
  disabled,
  loading,
  icon,
  children,
}: ButtonProps) {
  const theme = useTheme();

  const getMode = (variant: ButtonVariant) => {
    switch (variant) {
      case "primary":
      case "secondary":
        return "contained";
      case "outline":
        return "outlined";
      case "text":
        return "text";
      default:
        return "contained";
    }
  };

  const getButtonStyle = (variant: ButtonVariant, size: string) => {
    const baseStyle = {
      borderRadius: theme.roundness,
    };

    const sizeStyles = {
      small: {
        height: 32,
      },
      medium: {
        height: 40,
      },
      large: {
        height: 48,
      },
    };

    return [
      baseStyle,
      sizeStyles[size],
      variant === "secondary" && {
        backgroundColor: theme.colors.secondary.main,
      },
    ];
  };

  const getLabelStyle = (size: string) => {
    const sizes = {
      small: theme.typography.variants.labelMedium,
      medium: theme.typography.variants.labelLarge,
      large: theme.typography.variants.titleMedium,
    };
    return sizes[size];
  };

  return (
    <PaperButton
      mode={getMode(variant)}
      onPress={onPress}
      disabled={disabled}
      loading={loading}
      icon={icon}
      style={[styles.button, getButtonStyle(variant, size)]}
      labelStyle={getLabelStyle(size)}
      theme={{
        colors: {
          primary:
            variant === "secondary"
              ? theme.colors.secondary.main
              : theme.colors.primary.main,
        },
      }}
    >
      {children}
    </PaperButton>
  );
}

const styles = StyleSheet.create({
  button: {
    marginVertical: 8,
  },
});
```

Example of a Card component for mobile:

```typescript
// apps/mobile/src/components/JobCard/index.tsx
import { StyleSheet } from "react-native";
import { Card, Text, Chip, useTheme } from "react-native-paper";

interface JobCardProps {
  title: string;
  company: string;
  location: string;
  salary?: string;
  status: "applied" | "interviewing" | "offered" | "rejected";
  onPress: () => void;
}

export function JobCard({
  title,
  company,
  location,
  salary,
  status,
  onPress,
}: JobCardProps) {
  const theme = useTheme();

  const getStatusColor = (status: string) => {
    const colors = {
      applied: theme.colors.primary.main,
      interviewing: theme.colors.warning.main,
      offered: theme.colors.success.main,
      rejected: theme.colors.error.main,
    };
    return colors[status];
  };

  return (
    <Card style={styles.card} onPress={onPress} mode="elevated">
      <Card.Content style={styles.content}>
        <Text variant="titleLarge" style={styles.title}>
          {title}
        </Text>
        <Text variant="titleMedium" style={styles.company}>
          {company}
        </Text>
        <Text variant="bodyMedium" style={styles.location}>
          {location}
        </Text>
        {salary && (
          <Text variant="bodyMedium" style={styles.salary}>
            {salary}
          </Text>
        )}
        <Chip
          mode="flat"
          style={[
            styles.statusChip,
            { backgroundColor: getStatusColor(status) + "20" },
          ]}
          textStyle={{ color: getStatusColor(status) }}
        >
          {status.charAt(0).toUpperCase() + status.slice(1)}
        </Chip>
      </Card.Content>
    </Card>
  );
}

const styles = StyleSheet.create({
  card: {
    marginHorizontal: 16,
    marginVertical: 8,
  },
  content: {
    gap: 4,
  },
  title: {
    marginBottom: 4,
  },
  company: {
    opacity: 0.87,
  },
  location: {
    opacity: 0.6,
  },
  salary: {
    opacity: 0.6,
  },
  statusChip: {
    alignSelf: "flex-start",
    marginTop: 8,
  },
});
```

## Best Practices

1. **Theme Consistency**

   - Always use theme tokens (colors, typography, spacing)
   - Create semantic color names (e.g., `primary`, `error`)
   - Maintain consistent spacing using theme values

2. **Component Organization**

   - Keep components small and focused
   - Use composition for complex components
   - Create variants through props
   - Share common patterns between platforms when possible

3. **Style Organization**

   - Use platform-specific styling solutions (Tailwind for web, StyleSheet for mobile)
   - Keep styles close to components
   - Use consistent naming conventions
   - Follow platform conventions while maintaining visual consistency

4. **Accessibility**

   - Include proper accessibility labels
   - Maintain adequate touch targets (minimum 44x44 points for mobile)
   - Use semantic HTML elements on web
   - Support screen readers on both platforms
   - Ensure sufficient color contrast (WCAG 2.1 AA compliance)

5. **Cross-Platform Consistency**
   - Share design tokens between platforms
   - Maintain consistent component APIs where possible
   - Adapt to platform-specific patterns when necessary
   - Document platform-specific differences

## Common Patterns

### Form Layouts

```typescript
// apps/mobile/src/components/Form/FormLayout.tsx
import { StyleSheet, View } from "react-native";
import { useTheme } from "react-native-paper";

interface FormLayoutProps {
  children: React.ReactNode;
  spacing?: "small" | "medium" | "large";
}

export function FormLayout({ children, spacing = "medium" }: FormLayoutProps) {
  const theme = useTheme();

  const getSpacing = (size: string) => {
    const spacings = {
      small: theme.spacing.sm,
      medium: theme.spacing.md,
      large: theme.spacing.lg,
    };
    return spacings[size];
  };

  return (
    <View style={[styles.container, { gap: getSpacing(spacing) }]}>
      {children}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
  },
});
```

### List Items

```typescript
// apps/mobile/src/components/List/ListItem.tsx
import { StyleSheet } from "react-native";
import { List, useTheme } from "react-native-paper";

interface ListItemProps {
  title: string;
  description?: string;
  left?: (props: any) => React.ReactNode;
  right?: (props: any) => React.ReactNode;
  onPress?: () => void;
}

export function ListItem({
  title,
  description,
  left,
  right,
  onPress,
}: ListItemProps) {
  const theme = useTheme();

  return (
    <List.Item
      title={title}
      description={description}
      left={left}
      right={right}
      onPress={onPress}
      style={[styles.item, { backgroundColor: theme.colors.background.paper }]}
    />
  );
}

const styles = StyleSheet.create({
  item: {
    marginVertical: 1,
  },
});
```
