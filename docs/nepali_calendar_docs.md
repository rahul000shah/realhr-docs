# Nepali Calendar Widget Documentation

File -> `nepali_calendar.dart`
Path -> `real-hr-soft-app/lib/src/global_utils/widgets/nepali_calendar.dart`
## Overview

The Nepali Calendar widget is a custom Flutter widget that allows users to select dates using the Nepali calendar system. It supports date range selection and provides convenient preset filters like "Today," "This Week," "This Month," etc.

## Features

- Select single dates or date ranges
- Preset date filters (Today, This Week, This Month, Previous Month, This Year, All Time)
- Visual highlighting of selected dates and date ranges
- Option to hide certain UI elements
- Callback for date selection changes

## Usage

### Basic Usage

```dart
NepaliCalendar(
  onValuesUpdated: ({startDate, endDate, currentFilter, hasSelectedFiltersOnly}) {
    // Handle selected dates
    print('Start Date: $startDate');
    print('End Date: $endDate');
  }
)
```

### With Initial Values

```dart
NepaliCalendar(
  startDate: NepaliDateTime(2080, 1, 1),
  endDate: NepaliDateTime(2080, 1, 15),
  defaultFilter: DateFilters.thisMonth,
  onValuesUpdated: ({startDate, endDate, currentFilter, hasSelectedFiltersOnly}) {
    // Handle selected dates
  }
)
```

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `startDate` | `NepaliDateTime?` | Initial start date (optional) |
| `endDate` | `NepaliDateTime?` | Initial end date (optional) |
| `defaultFilter` | `DateFilters?` | Default date filter to apply (optional) |
| `hasSelectedFilterOnly` | `bool` | Whether to only show the selected filter (default: true) |
| `hideTopBarOnly` | `bool` | Whether to hide the top bar (default: false) |
| `onValuesUpdated` | `Function?` | Callback function that triggers when dates are updated |

## Date Filters

The widget supports the following date filter options:

- **Today**: Current day only
- **This Week**: Current week (adjusted to stay within current month)
- **This Month**: From the 1st to the last day of the current month
- **Previous Month**: From the 1st to the last day of the previous month
- **This Year**: From January 1st to December 31st of the current year
- **All Time**: No date restrictions

## Visual Elements

The calendar contains these main visual components:

1. **Top Bar** (optional)
   - "Cancel" button to dismiss the calendar
   - Date range display text
   - "Done" button to confirm selection

2. **Filter Bar**
   - Horizontally scrollable row of filter options

3. **Calendar Pickers**
   - Two calendar views for selecting start and end dates
   - Visual highlighting of selected date range

## Return Values

When the user taps "Done," the widget returns an array with:
1. Start date (as DateTime)
2. End date (as DateTime)
3. Current filter (DateFilters enum)
4. Boolean indicating if a predefined filter is selected

## Key Functions

- `_setDate(DateFilters value)`: Sets the date range based on the selected filter
- `_handleStartDateChange(NepaliDateTime value)`: Updates the start date
- `_handleEndDateChange(NepaliDateTime value)`: Updates the end date
- `_updateFilterBasedOnDates()`: Determines which filter (if any) matches the selected dates

## Styling

The widget uses the app's theme colors and custom styling for:
- Selected dates (highlighted with the primary color)
- Date range (lighter shade of the primary color)
- Today's date (special highlighting when not selected)
- Buttons and text elements

## Date Range Color Mapping

The calendar uses a smart color mapping system to visually represent date ranges. Here's how it works:

### In the `dayBuilder` Method

The `dayBuilder` method in the `_buildCalendarPicker` function handles all the visual styling for each day cell:

1. **Determining Date Status**:
   - `isSelected`: Checks if the current date matches the selected date
   - `isStartDateMatch`: Checks if the date matches the start date
   - `isEndDateMatch`: Checks if the date matches the end date
   - `isInRange`: Checks if the date falls between start and end dates
   - `isToday`: Checks if the date is today

2. **Color Application Logic**:
   - **Edge Dates** (start or end date): Circular shape with primary color background and white text
   - **In-Range Dates**: Rectangular shape with semi-transparent primary color (opacity 0.2)
   - **Today's Date** (if not selected): Regular color with primary color text

3. **Edge Date Indicators**:
   - When a date is either the start or end of a range, a half-colored background extends toward the range
   - Start date: Right half extension
   - End date: Left half extension

4. **Code Implementation**:
   ```dart
   // Determining if day should show color
   bool shouldShowColor = false;
   if (isStartDate) {
     shouldShowColor = isStartDateMatch || (isInRange && !isEndDateMatch);
   } else {
     shouldShowColor = isEndDateMatch || (isInRange && !isStartDateMatch);
   }

   // Setting background color based on date status
   final Color backgroundColor = shouldShowColor
       ? (isEdgeDate)
           ? Theme.of(context).primaryColor
           : isInRange
               ? Theme.of(context).primaryColor.withOpacity(0.2)
               : Colors.transparent
       : Colors.transparent;
   ```

5. **Range Visualization**:
   - For dates at the range edges, a half-colored rectangle extends toward the range
   - This creates a visual "bridge" between selected dates
   - Implemented using a Stack with alignment to create the half-colored effect

This color mapping system creates a clear visual distinction between:
- Selected edge dates (start/end dates)
- Dates within the selected range
- Dates outside the selected range
- The current day (when not selected)

## Dependencies

This widget requires:
- `nepali_date_picker: ^6.0.1` package

## Dependencies Override
  `nepali_utils: 3.0.8` package-
