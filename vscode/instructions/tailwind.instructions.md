---
description: "Tailwind CSS and UI component best practices for modern web applications. Cre: https://github.com/PatrickJS/awesome-cursorrules/blob/main/rules-new/tailwind.mdc"
applyTo: "**/*.jsx, **/*.tsx, tailwind.config.js, tailwind.config.ts"
---

# Tailwind CSS Best Practices

Instructions for building high-quality web applications with Tailwind CSS, following modern design patterns, responsive design principles, and accessibility guidelines.

## Project Setup

- You MUST configure Tailwind CSS with proper theme extensions and custom properties.
  Reasoning: Proper configuration ensures consistent design tokens, enables customization, and maintains design system scalability.

  Example:

  ```js
  // tailwind.config.js
  module.exports = {
    content: ["./src/**/*.{js,ts,jsx,tsx}"],
    theme: {
      extend: {
        colors: {
          primary: {
            50: "#eff6ff",
            500: "#3b82f6",
            900: "#1e3a8a",
          },
          secondary: {
            50: "#f8fafc",
            500: "#64748b",
            900: "#0f172a",
          },
        },
        spacing: {
          18: "4.5rem",
          88: "22rem",
        },
        fontFamily: {
          sans: ["Inter", "system-ui", "sans-serif"],
        },
      },
    },
    plugins: [
      require("@tailwindcss/forms"),
      require("@tailwindcss/typography"),
    ],
  };
  ```

- You MUST set up proper content paths for purge optimization in production builds.
  Reasoning: Content paths ensure unused styles are removed in production, reducing bundle size and improving performance.

  Example:

  ```js
  // tailwind.config.js
  module.exports = {
    content: [
      "./src/**/*.{js,ts,jsx,tsx}",
      "./public/index.html",
      "./node_modules/@your-org/**/*.{js,ts,jsx,tsx}",
    ],
    // ...
  };
  ```

- You MUST configure custom breakpoints and container queries when needed.
  Reasoning: Custom breakpoints and container queries enable more precise responsive design and component-based layouts.

  Example:

  ```js
  // tailwind.config.js
  module.exports = {
    theme: {
      extend: {
        screens: {
          xs: "475px",
          "3xl": "1600px",
        },
        container: {
          center: true,
          padding: {
            DEFAULT: "1rem",
            sm: "2rem",
            lg: "4rem",
            xl: "5rem",
            "2xl": "6rem",
          },
        },
      },
    },
  };
  ```

## Component Styling

- You MUST use utility classes over custom CSS for consistent styling and maintainability.
  Reasoning: Utility classes provide consistent design tokens, reduce CSS bundle size, and make styling more predictable and maintainable.

  Example:

  ```tsx
  // Good: Using utility classes
  function Button({
    children,
    variant = "primary",
  }: {
    children: React.ReactNode;
    variant?: "primary" | "secondary";
  }) {
    const baseClasses =
      "px-4 py-2 rounded-lg font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2";
    const variantClasses = {
      primary: "bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500",
      secondary:
        "bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-500",
    };

    return (
      <button className={`${baseClasses} ${variantClasses[variant]}`}>
        {children}
      </button>
    );
  }

  // Bad: Custom CSS
  function BadButton({ children }: { children: React.ReactNode }) {
    return <button className="custom-button">{children}</button>;
  }
  ```

- You MUST group related utilities with @apply when creating reusable component styles.
  Reasoning: @apply directive helps organize complex utility combinations into readable, reusable styles while maintaining the benefits of utility-first CSS.

  Example:

  ```css
  /* components.css */
  .btn-primary {
    @apply px-4 py-2 bg-blue-600 text-white rounded-lg font-medium;
    @apply hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2;
    @apply transition-colors duration-200;
  }

  .card {
    @apply bg-white rounded-lg shadow-md p-6 border border-gray-200;
    @apply hover:shadow-lg transition-shadow duration-200;
  }
  ```

- You MUST implement responsive design using mobile-first approach with Tailwind's responsive prefixes.
  Reasoning: Mobile-first responsive design ensures better performance and user experience across all device sizes.

  Example:

  ```tsx
  function ResponsiveCard({ children }: { children: React.ReactNode }) {
    return (
      <div
        className="
        w-full p-4 mx-2
        sm:w-auto sm:mx-4 sm:p-6
        md:mx-6 md:p-8
        lg:mx-8 lg:p-10
      "
      >
        {children}
      </div>
    );
  }
  ```

- You MUST implement dark mode using Tailwind's dark mode utilities and proper color schemes.
  Reasoning: Dark mode improves user experience and accessibility, especially in low-light environments.

  Example:

  ```tsx
  function ThemeAwareCard({ children }: { children: React.ReactNode }) {
    return (
      <div
        className="
        bg-white dark:bg-gray-800
        text-gray-900 dark:text-gray-100
        border border-gray-200 dark:border-gray-700
        shadow-sm dark:shadow-gray-900/20
        rounded-lg p-6
      "
      >
        {children}
      </div>
    );
  }
  ```

- You MUST use proper state variants (hover, focus, active, disabled) for interactive elements.
  Reasoning: State variants provide visual feedback to users, improving accessibility and user experience.

  Example:

  ```tsx
  function InteractiveButton({
    children,
    disabled,
  }: {
    children: React.ReactNode;
    disabled?: boolean;
  }) {
    return (
      <button
        disabled={disabled}
        className="
          px-4 py-2 bg-blue-600 text-white rounded-lg font-medium
          hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
          active:bg-blue-800 disabled:opacity-50 disabled:cursor-not-allowed
          transition-colors duration-200
        "
      >
        {children}
      </button>
    );
  }
  ```

## Layout

- You MUST use Flexbox and Grid utilities effectively for modern layout patterns.
  Reasoning: Flexbox and Grid provide powerful, flexible layout capabilities that work well across different screen sizes and content types.

  Example:

  ```tsx
  // Flexbox layout
  function FlexLayout({ children }: { children: React.ReactNode }) {
    return (
      <div className="flex flex-col sm:flex-row gap-4 items-center justify-between">
        {children}
      </div>
    );
  }

  // Grid layout
  function GridLayout({ children }: { children: React.ReactNode }) {
    return (
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
        {children}
      </div>
    );
  }
  ```

- You MUST implement proper spacing using Tailwind's spacing scale for consistency.
  Reasoning: Consistent spacing creates visual hierarchy, improves readability, and maintains design system coherence.

  Example:

  ```tsx
  function ConsistentSpacing({ children }: { children: React.ReactNode }) {
    return (
      <div className="space-y-4 p-6">
        <h2 className="text-xl font-semibold mb-4">Title</h2>
        <div className="space-y-2">{children}</div>
        <div className="mt-6 pt-4 border-t border-gray-200">Footer content</div>
      </div>
    );
  }
  ```

- You MUST use container queries when components need to adapt to their container size.
  Reasoning: Container queries enable components to respond to their container's size rather than just the viewport, creating more flexible and reusable components.

  Example:

  ```tsx
  function ContainerResponsiveCard({
    children,
  }: {
    children: React.ReactNode;
  }) {
    return (
      <div
        className="
        @container
        bg-white rounded-lg shadow-md p-4
        @sm:p-6 @lg:p-8
        @sm:grid @sm:grid-cols-2 @sm:gap-4
        @lg:grid-cols-3 @lg:gap-6
      "
      >
        {children}
      </div>
    );
  }
  ```

- You MUST implement proper responsive breakpoints using Tailwind's default or custom breakpoints.
  Reasoning: Proper breakpoints ensure optimal layout and user experience across all device sizes.

  Example:

  ```tsx
  function ResponsiveLayout({ children }: { children: React.ReactNode }) {
    return (
      <div
        className="
        w-full max-w-7xl mx-auto px-4
        sm:px-6 lg:px-8
        grid grid-cols-1
        sm:grid-cols-2 lg:grid-cols-3
        gap-4 sm:gap-6 lg:gap-8
      "
      >
        {children}
      </div>
    );
  }
  ```

## Typography

- You MUST use Tailwind's font size utilities for consistent text scaling.
  Reasoning: Consistent font sizes create visual hierarchy and improve readability across the application.

  Example:

  ```tsx
  function TypographyExample() {
    return (
      <div className="space-y-4">
        <h1 className="text-4xl font-bold text-gray-900">Main Heading</h1>
        <h2 className="text-2xl font-semibold text-gray-800">
          Section Heading
        </h2>
        <h3 className="text-xl font-medium text-gray-700">Subsection</h3>
        <p className="text-base text-gray-600 leading-relaxed">
          Body text with proper line height.
        </p>
        <small className="text-sm text-gray-500">
          Small text for captions.
        </small>
      </div>
    );
  }
  ```

- You MUST implement proper line height for optimal readability.
  Reasoning: Appropriate line height improves text readability and visual appeal, especially for longer content.

  Example:

  ```tsx
  function ReadableText({ children }: { children: React.ReactNode }) {
    return (
      <div
        className="
        text-base leading-relaxed
        sm:text-lg sm:leading-loose
        lg:text-xl lg:leading-loose
        text-gray-700
      "
      >
        {children}
      </div>
    );
  }
  ```

- You MUST use semantic font weight utilities for proper text hierarchy.
  Reasoning: Semantic font weights create clear visual hierarchy and improve content scannability.

  Example:

  ```tsx
  function TextHierarchy() {
    return (
      <div className="space-y-2">
        <h1 className="text-3xl font-bold text-gray-900">Bold Heading</h1>
        <h2 className="text-2xl font-semibold text-gray-800">
          Semibold Subheading
        </h2>
        <h3 className="text-xl font-medium text-gray-700">Medium Title</h3>
        <p className="text-base font-normal text-gray-600">Normal body text</p>
        <span className="text-sm font-light text-gray-500">Light caption</span>
      </div>
    );
  }
  ```

- You MUST configure custom fonts properly in the Tailwind config.
  Reasoning: Custom fonts enhance brand identity and improve typography consistency across the application.

  Example:

  ```js
  // tailwind.config.js
  module.exports = {
    theme: {
      extend: {
        fontFamily: {
          sans: ["Inter", "system-ui", "sans-serif"],
          serif: ["Georgia", "serif"],
          mono: ["JetBrains Mono", "monospace"],
          display: ["Poppins", "sans-serif"],
        },
      },
    },
  };
  ```

## Colors

- You MUST use semantic color naming for better maintainability and accessibility.
  Reasoning: Semantic color names make code more readable and easier to maintain, especially when updating design systems.

  Example:

  ```tsx
  function SemanticColors() {
    return (
      <div className="space-y-4">
        <div className="bg-primary-500 text-primary-50 p-4 rounded">
          Primary action button
        </div>
        <div className="bg-success-500 text-white p-4 rounded">
          Success message
        </div>
        <div className="bg-warning-500 text-warning-900 p-4 rounded">
          Warning alert
        </div>
        <div className="bg-error-500 text-white p-4 rounded">Error message</div>
      </div>
    );
  }
  ```

- You MUST implement proper color contrast ratios for accessibility compliance.
  Reasoning: Adequate color contrast ensures text is readable by all users, including those with visual impairments.

  Example:

  ```tsx
  function AccessibleText() {
    return (
      <div className="space-y-4">
        {/* Good contrast: dark text on light background */}
        <p className="text-gray-900 bg-gray-50 p-4 rounded">
          High contrast text for accessibility
        </p>

        {/* Good contrast: light text on dark background */}
        <p className="text-gray-100 bg-gray-800 p-4 rounded">
          High contrast text on dark background
        </p>

        {/* Avoid: low contrast combinations */}
        {/* <p className="text-gray-400 bg-gray-300">Hard to read</p> */}
      </div>
    );
  }
  ```

- You MUST use opacity utilities effectively for overlays and subtle effects.
  Reasoning: Opacity utilities create depth, improve visual hierarchy, and enable sophisticated design patterns.

  Example:

  ```tsx
  function OpacityEffects() {
    return (
      <div className="relative">
        <img src="/hero.jpg" alt="Hero" className="w-full h-64 object-cover" />
        <div className="absolute inset-0 bg-black bg-opacity-50 flex items-center justify-center">
          <h1 className="text-white text-3xl font-bold">Overlay Text</h1>
        </div>

        <div className="mt-4 p-4 bg-blue-500 bg-opacity-10 border border-blue-200 rounded">
          Subtle background with opacity
        </div>
      </div>
    );
  }
  ```

- You MUST configure custom colors properly in the Tailwind config for brand consistency.
  Reasoning: Custom colors ensure brand consistency and provide a cohesive design system across the application.

  Example:

  ```js
  // tailwind.config.js
  module.exports = {
    theme: {
      extend: {
        colors: {
          brand: {
            50: "#f0f9ff",
            100: "#e0f2fe",
            500: "#0ea5e9",
            600: "#0284c7",
            700: "#0369a1",
            900: "#0c4a6e",
          },
          success: {
            50: "#f0fdf4",
            500: "#22c55e",
            600: "#16a34a",
            700: "#15803d",
          },
          warning: {
            50: "#fffbeb",
            500: "#f59e0b",
            600: "#d97706",
            700: "#b45309",
          },
          error: {
            50: "#fef2f2",
            500: "#ef4444",
            600: "#dc2626",
            700: "#b91c1c",
          },
        },
      },
    },
  };
  ```

## Components

- You MUST use shadcn/ui components when available for consistent, accessible UI patterns.
  Reasoning: shadcn/ui provides well-tested, accessible components that follow modern design patterns and reduce development time.

  Example:

  ```tsx
  import { Button } from "@/components/ui/button";
  import {
    Card,
    CardContent,
    CardHeader,
    CardTitle,
  } from "@/components/ui/card";
  import { Input } from "@/components/ui/input";

  function ShadcnExample() {
    return (
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle>User Profile</CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <Input placeholder="Enter your name" />
          <Button className="w-full">Save Changes</Button>
        </CardContent>
      </Card>
    );
  }
  ```

- You MUST extend components properly using composition and variant patterns.
  Reasoning: Proper component extension enables reusability while maintaining flexibility and customization options.

  Example:

  ```tsx
  import { cva, type VariantProps } from "class-variance-authority";
  import { cn } from "@/lib/utils";

  const buttonVariants = cva(
    "inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background",
    {
      variants: {
        variant: {
          default: "bg-primary text-primary-foreground hover:bg-primary/90",
          destructive:
            "bg-destructive text-destructive-foreground hover:bg-destructive/90",
          outline:
            "border border-input hover:bg-accent hover:text-accent-foreground",
          secondary:
            "bg-secondary text-secondary-foreground hover:bg-secondary/80",
          ghost: "hover:bg-accent hover:text-accent-foreground",
          link: "underline-offset-4 hover:underline text-primary",
        },
        size: {
          default: "h-10 py-2 px-4",
          sm: "h-9 px-3 rounded-md",
          lg: "h-11 px-8 rounded-md",
          icon: "h-10 w-10",
        },
      },
      defaultVariants: {
        variant: "default",
        size: "default",
      },
    }
  );

  interface ButtonProps
    extends React.ButtonHTMLAttributes<HTMLButtonElement>,
      VariantProps<typeof buttonVariants> {}

  function Button({ className, variant, size, ...props }: ButtonProps) {
    return (
      <button
        className={cn(buttonVariants({ variant, size, className }))}
        {...props}
      />
    );
  }
  ```

- You MUST implement proper animations using Tailwind's transition utilities.
  Reasoning: Smooth animations improve user experience and provide visual feedback for interactions.

  Example:

  ```tsx
  function AnimatedComponents() {
    return (
      <div className="space-y-4">
        <button
          className="
          px-4 py-2 bg-blue-600 text-white rounded-lg
          hover:bg-blue-700 active:bg-blue-800
          transform hover:scale-105 active:scale-95
          transition-all duration-200 ease-in-out
        "
        >
          Animated Button
        </button>

        <div
          className="
          p-4 bg-white rounded-lg shadow-md
          hover:shadow-lg hover:-translate-y-1
          transition-all duration-300 ease-out
        "
        >
          Hover to lift
        </div>
      </div>
    );
  }
  ```

## Responsive Design

- You MUST use mobile-first approach with Tailwind's responsive prefixes.
  Reasoning: Mobile-first design ensures optimal performance and user experience across all devices, starting with the most constrained environment.

  Example:

  ```tsx
  function MobileFirstLayout() {
    return (
      <div
        className="
        grid grid-cols-1 gap-4 p-4
        sm:grid-cols-2 sm:gap-6 sm:p-6
        md:grid-cols-3 md:gap-8 md:p-8
        lg:grid-cols-4 lg:gap-10 lg:p-10
      "
      >
        {/* Content */}
      </div>
    );
  }
  ```

- You MUST handle different screen sizes properly with appropriate breakpoints.
  Reasoning: Proper breakpoint handling ensures content is optimally displayed across all device sizes and orientations.

  Example:

  ```tsx
  function ResponsiveNavigation() {
    return (
      <nav
        className="
        flex flex-col space-y-2 p-4
        sm:flex-row sm:space-y-0 sm:space-x-4 sm:p-6
        lg:space-x-8 lg:p-8
      "
      >
        <a
          href="/"
          className="text-lg font-medium hover:text-blue-600 transition-colors"
        >
          Home
        </a>
        <a
          href="/about"
          className="text-lg font-medium hover:text-blue-600 transition-colors"
        >
          About
        </a>
        <a
          href="/contact"
          className="text-lg font-medium hover:text-blue-600 transition-colors"
        >
          Contact
        </a>
      </nav>
    );
  }
  ```

- You MUST implement proper responsive typography for optimal readability.
  Reasoning: Responsive typography ensures text remains readable and appropriately sized across all devices.

  Example:

  ```tsx
  function ResponsiveTypography() {
    return (
      <div className="space-y-4">
        <h1
          className="
          text-2xl font-bold text-gray-900
          sm:text-3xl md:text-4xl lg:text-5xl
        "
        >
          Responsive Heading
        </h1>

        <p
          className="
          text-sm leading-relaxed text-gray-600
          sm:text-base sm:leading-loose
          lg:text-lg lg:leading-loose
        "
        >
          This paragraph scales appropriately across different screen sizes
          while maintaining optimal line length and readability.
        </p>
      </div>
    );
  }
  ```

## Performance

- You MUST use proper purge configuration to remove unused styles in production.
  Reasoning: Purge optimization significantly reduces CSS bundle size, improving load times and performance.

  Example:

  ```js
  // tailwind.config.js
  module.exports = {
    content: [
      "./src/**/*.{js,ts,jsx,tsx}",
      "./public/index.html",
      "./node_modules/@your-org/**/*.{js,ts,jsx,tsx}",
    ],
    // Purge is enabled by default in Tailwind CSS v3+
  };
  ```

- You MUST minimize custom CSS in favor of utility classes.
  Reasoning: Utility classes reduce CSS bundle size, improve maintainability, and provide consistent design tokens.

  Example:

  ```tsx
  // Good: Utility classes only
  function GoodComponent() {
    return (
      <div className="bg-white rounded-lg shadow-md p-6 border border-gray-200">
        <h2 className="text-xl font-semibold text-gray-900 mb-4">Title</h2>
        <p className="text-gray-600 leading-relaxed">Content</p>
      </div>
    );
  }

  // Bad: Custom CSS
  function BadComponent() {
    return (
      <div className="custom-card">
        <h2 className="custom-title">Title</h2>
        <p className="custom-text">Content</p>
      </div>
    );
  }
  ```

- You MUST use proper caching strategies for CSS assets.
  Reasoning: CSS caching reduces load times for returning users and improves overall application performance.

  Example:

  ```html
  <!-- In your HTML head -->
  <link rel="stylesheet" href="/styles.css?v=1.0.0" />
  ```

- You MUST optimize for production by enabling all Tailwind optimizations.
  Reasoning: Production optimizations ensure the smallest possible bundle size and best performance for end users.

  Example:

  ```js
  // tailwind.config.js
  module.exports = {
    content: ["./src/**/*.{js,ts,jsx,tsx}"],
    theme: {
      extend: {
        // Custom theme extensions
      },
    },
    plugins: [
      // Only include plugins you actually use
      require("@tailwindcss/forms"),
      require("@tailwindcss/typography"),
    ],
    // Enable all optimizations
    future: {
      hoverOnlyWhenSupported: true,
    },
  };
  ```

## Best Practices

- You MUST follow consistent naming conventions for utility classes.
  Reasoning: Consistent naming improves code readability, maintainability, and team collaboration.

  Example:

  ```tsx
  // Good: Consistent spacing and organization
  function WellNamedComponent() {
    return (
      <div
        className="
        flex flex-col items-center justify-center
        w-full max-w-md mx-auto
        p-6 space-y-4
        bg-white rounded-lg shadow-md
        border border-gray-200
      "
      >
        <h2 className="text-xl font-semibold text-gray-900">Title</h2>
        <p className="text-gray-600 text-center">Description</p>
      </div>
    );
  }
  ```

- You MUST keep styles organized and maintainable.
  Reasoning: Organized styles improve code readability, reduce debugging time, and make refactoring easier.

  Example:

  ```tsx
  function OrganizedComponent() {
    // Group related utilities with line breaks and comments
    const containerClasses = `
      flex flex-col items-center justify-center
      w-full max-w-md mx-auto
      p-6 space-y-4
    `;

    const visualClasses = `
      bg-white rounded-lg shadow-md
      border border-gray-200
      hover:shadow-lg transition-shadow duration-200
    `;

    const textClasses = `
      text-xl font-semibold text-gray-900
      dark:text-gray-100
    `;

    return (
      <div className={`${containerClasses} ${visualClasses}`}>
        <h2 className={textClasses}>Title</h2>
        <p className="text-gray-600 dark:text-gray-300 text-center">
          Description
        </p>
      </div>
    );
  }
  ```

- You MUST implement proper accessibility guidelines with Tailwind utilities.
  Reasoning: Accessibility ensures your application is usable by all users, including those with disabilities, and complies with legal requirements.

  Example:

  ```tsx
  function AccessibleComponent() {
    return (
      <div className="space-y-4">
        {/* Proper focus states */}
        <button
          className="
          px-4 py-2 bg-blue-600 text-white rounded-lg
          hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
          transition-colors duration-200
        "
        >
          Accessible Button
        </button>

        {/* High contrast text */}
        <p className="text-gray-900 bg-gray-50 p-4 rounded">
          High contrast text for accessibility
        </p>

        {/* Proper spacing for touch targets */}
        <button
          className="
          w-12 h-12 bg-gray-200 rounded-lg
          hover:bg-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-500
          transition-colors duration-200
        "
        >
          Touch Target
        </button>
      </div>
    );
  }
  ```

- You MUST use proper version control practices for CSS changes.
  Reasoning: Version control practices ensure changes are tracked, can be reviewed, and can be reverted if needed.

  Example:

  ```bash
  # Good commit messages for CSS changes
  git commit -m "feat: add responsive card component with Tailwind utilities"
  git commit -m "fix: improve button accessibility with proper focus states"
  git commit -m "refactor: consolidate color utilities for better maintainability"
  ```

## Implementation Process

1. Set up Tailwind CSS with proper configuration and content paths.
2. Define custom theme extensions for colors, spacing, and typography.
3. Create base component styles using utility classes.
4. Implement responsive design patterns with mobile-first approach.
5. Add dark mode support using Tailwind's dark mode utilities.
6. Create reusable component variants using class-variance-authority.
7. Implement proper accessibility features with focus states and contrast.
8. Optimize for production with purge configuration.
9. Test across different screen sizes and devices.
10. Document component usage and styling patterns.

## Additional Guidelines

- Use Tailwind CSS IntelliSense extension for better development experience
- Leverage Tailwind's arbitrary value syntax for one-off customizations
- Use CSS custom properties with Tailwind for dynamic theming
- Implement proper loading states and skeleton screens
- Use Tailwind's aspect ratio utilities for responsive images
- Leverage Tailwind's backdrop utilities for modern overlay effects
- Use Tailwind's scroll utilities for custom scrollbar styling
- Implement proper print styles using Tailwind's print utilities

## Patterns

- Utility-first CSS for consistent and maintainable styles
- Component composition for flexible and reusable UI patterns
- Responsive design with mobile-first approach
- Dark mode implementation using CSS custom properties
- Accessibility-first design with proper focus states and contrast
- Performance optimization through purge and caching strategies

---

<!-- End of Tailwind Best Practices instructions -->
