# TypeScript Rules & Best Practices

---

## Date & Time Formatting

### Using Intl.DateTimeFormat for US-Based Timestamps

`Intl.DateTimeFormat` is the modern, locale-aware way to format dates and times in JavaScript/TypeScript. For US-based timestamps, use the `'en-US'` locale.

#### Basic Usage

```typescript
// Create a formatter for US locale
const formatter = new Intl.DateTimeFormat("en-US", {
  year: "numeric",
  month: "long",
  day: "numeric",
  hour: "numeric",
  minute: "2-digit",
  hour12: true, // 12-hour format (default for en-US)
});

const date = new Date();
const formatted = formatter.format(date);
// Example output: "December 25, 2023, 3:45 PM"
```

#### Common US Date/Time Formats

```typescript
// Short date (MM/DD/YYYY)
const shortDate = new Intl.DateTimeFormat("en-US", {
  month: "2-digit",
  day: "2-digit",
  year: "numeric",
});
shortDate.format(new Date()); // "12/25/2023"

// Long date (Month DD, YYYY)
const longDate = new Intl.DateTimeFormat("en-US", {
  month: "long",
  day: "numeric",
  year: "numeric",
});
longDate.format(new Date()); // "December 25, 2023"

// Date and time (12-hour format)
const dateTime = new Intl.DateTimeFormat("en-US", {
  month: "2-digit",
  day: "2-digit",
  year: "numeric",
  hour: "numeric",
  minute: "2-digit",
  hour12: true,
});
dateTime.format(new Date()); // "12/25/2023, 3:45 PM"

// Date and time (24-hour format)
const dateTime24 = new Intl.DateTimeFormat("en-US", {
  month: "2-digit",
  day: "2-digit",
  year: "numeric",
  hour: "2-digit",
  minute: "2-digit",
  hour12: false,
});
dateTime24.format(new Date()); // "12/25/2023, 15:45"

// Time only (12-hour)
const time12 = new Intl.DateTimeFormat("en-US", {
  hour: "numeric",
  minute: "2-digit",
  second: "2-digit",
  hour12: true,
});
time12.format(new Date()); // "3:45:30 PM"

// Time only (24-hour)
const time24 = new Intl.DateTimeFormat("en-US", {
  hour: "2-digit",
  minute: "2-digit",
  second: "2-digit",
  hour12: false,
});
time24.format(new Date()); // "15:45:30"

// ISO-like format (YYYY-MM-DD HH:MM:SS)
const isoLike = new Intl.DateTimeFormat("en-US", {
  year: "numeric",
  month: "2-digit",
  day: "2-digit",
  hour: "2-digit",
  minute: "2-digit",
  second: "2-digit",
  hour12: false,
  timeZone: "America/New_York", // Optional: specify timezone
});
isoLike.format(new Date()); // "12/25/2023, 15:45:30"
```

#### Format Options Reference

| Option         | Values                                                    | Description               |
| -------------- | --------------------------------------------------------- | ------------------------- |
| `year`         | `'numeric'`, `'2-digit'`                                  | Year representation       |
| `month`        | `'numeric'`, `'2-digit'`, `'long'`, `'short'`, `'narrow'` | Month representation      |
| `day`          | `'numeric'`, `'2-digit'`                                  | Day of month              |
| `weekday`      | `'long'`, `'short'`, `'narrow'`                           | Day of week               |
| `hour`         | `'numeric'`, `'2-digit'`                                  | Hour (1-12 or 0-23)       |
| `minute`       | `'numeric'`, `'2-digit'`                                  | Minute                    |
| `second`       | `'numeric'`, `'2-digit'`                                  | Second                    |
| `hour12`       | `true`, `false`                                           | 12-hour vs 24-hour format |
| `timeZone`     | `'America/New_York'`, `'America/Los_Angeles'`, etc.       | Timezone                  |
| `timeZoneName` | `'short'`, `'long'`                                       | Show timezone name        |

#### Timezone Handling

```typescript
// Eastern Time
const etFormatter = new Intl.DateTimeFormat("en-US", {
  year: "numeric",
  month: "2-digit",
  day: "2-digit",
  hour: "numeric",
  minute: "2-digit",
  hour12: true,
  timeZone: "America/New_York",
});

// Pacific Time
const ptFormatter = new Intl.DateTimeFormat("en-US", {
  year: "numeric",
  month: "2-digit",
  day: "2-digit",
  hour: "numeric",
  minute: "2-digit",
  hour12: true,
  timeZone: "America/Los_Angeles",
});

// Central Time
const ctFormatter = new Intl.DateTimeFormat("en-US", {
  timeZone: "America/Chicago",
  // ... other options
});
```

#### Reusable Formatter Functions

```typescript
// Create reusable formatters
export const USDateFormatters = {
  short: new Intl.DateTimeFormat("en-US", {
    month: "2-digit",
    day: "2-digit",
    year: "numeric",
  }),

  long: new Intl.DateTimeFormat("en-US", {
    month: "long",
    day: "numeric",
    year: "numeric",
  }),

  dateTime: new Intl.DateTimeFormat("en-US", {
    month: "2-digit",
    day: "2-digit",
    year: "numeric",
    hour: "numeric",
    minute: "2-digit",
    hour12: true,
  }),

  timeOnly: new Intl.DateTimeFormat("en-US", {
    hour: "numeric",
    minute: "2-digit",
    hour12: true,
  }),
};

// Usage
USDateFormatters.short.format(new Date()); // "12/25/2023"
USDateFormatters.long.format(new Date()); // "December 25, 2023"
```

#### Formatting Specific Dates

```typescript
// Format a specific date
const specificDate = new Date("2023-12-25T15:45:30Z");
const formatter = new Intl.DateTimeFormat("en-US", {
  month: "long",
  day: "numeric",
  year: "numeric",
  hour: "numeric",
  minute: "2-digit",
  hour12: true,
});
formatter.format(specificDate); // "December 25, 2023, 3:45 PM"

// Format Unix timestamp
const timestamp = 1703521530000; // milliseconds
const dateFromTimestamp = new Date(timestamp);
formatter.format(dateFromTimestamp);
```

#### Best Practices

1. **Create formatters once and reuse** - Don't create new formatters in loops
2. **Specify timezone explicitly** - Use `timeZone` option for consistency
3. **Use appropriate precision** - Use `'2-digit'` for consistent padding
4. **Consider user preferences** - For user-facing dates, consider detecting locale
5. **Handle invalid dates** - Always validate dates before formatting

```typescript
// Good: Reusable formatter
const formatter = new Intl.DateTimeFormat("en-US", {
  /* options */
});

function formatDate(date: Date): string {
  if (isNaN(date.getTime())) {
    throw new Error("Invalid date");
  }
  return formatter.format(date);
}

// Bad: Creating formatter repeatedly
function formatDateBad(date: Date): string {
  return new Intl.DateTimeFormat("en-US", {
    /* options */
  }).format(date);
}
```

#### Common US Timezone Identifiers

```typescript
"America/New_York"; // Eastern Time (ET)
"America/Chicago"; // Central Time (CT)
"America/Denver"; // Mountain Time (MT)
"America/Los_Angeles"; // Pacific Time (PT)
"America/Anchorage"; // Alaska Time (AKT)
"America/Honolulu"; // Hawaii Time (HST)
```

#### TypeScript Type Safety

```typescript
// Type-safe formatter function
type DateFormatOptions = {
  includeTime?: boolean;
  includeSeconds?: boolean;
  hour12?: boolean;
  timeZone?: string;
};

function createUSFormatter(
  options: DateFormatOptions = {}
): Intl.DateTimeFormat {
  const formatOptions: Intl.DateTimeFormatOptions = {
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
    hour12: options.hour12 ?? true,
    timeZone: options.timeZone ?? "America/New_York",
  };

  if (options.includeTime) {
    formatOptions.hour = "numeric";
    formatOptions.minute = "2-digit";

    if (options.includeSeconds) {
      formatOptions.second = "2-digit";
    }
  }

  return new Intl.DateTimeFormat("en-US", formatOptions);
}

// Usage
const formatter = createUSFormatter({
  includeTime: true,
  hour12: false,
  timeZone: "America/Los_Angeles",
});
```
