# TimeSheet Cell Color Documentation

## Function: `getCellColor`

Path : `real-hr-soft-app/lib/src/features/supervisor/features/supervisor_attendance/attendance_reports/utils/get_color.dart`

### Purpose
The `getCellColor` function determines the appropriate color code for a timesheet cell based on various conditions related to the employee's attendance status, punch records, and leave details.


### Function Call
```dart
Color? getCellColor({
  required String label, 
  required TimeSheetResult timeSheet
})
```

### Parameters
- `label` (String): The category label for the day (e.g., 'Holiday', 'Offday', 'Workday').
- `timeSheet` (TimeSheetResult): An object containing the employee's timesheet data for a specific day.

### Return Value
- `Color`: A Flutter Color object representing the appropriate color for visual representation.
- `null`: Returned if no specific color code applies (default case).

### Color Codes and Their Meanings

| Status | Color | Hex Code | Description |
|--------|-------|----------|-------------|
| Holiday | Blue | `0xFF2196F3` | Official holidays |
| Offday | Grey | `0xFF9E9E9E` | Regular days off (e.g., weekends) |
| Long-term Absence | Red | `0xFFF44336` | Employee marked as long-term absent (`[LR] Absent`) |
| Absence | Orange | `0xFFFF9800` | Employee marked as absent (but not long-term) |
| Missing Punch In/Out | Red | `0xFFF44336` | Missing punch-in or punch-out records with 'Missing' or 'N/A' indicators |
| Leave | Purple | `0xFF9C27B0` | Employee on approved leave |
| Early Out | Light Red | `0xFFE57373` | Employee punched out earlier than scheduled (negative punch-out delta) |
| On-time or Early Arrival | Green | `0xFF4CAF50` | Employee arrived on time or was late (negative punch-in delta) |
| Perfect Attendance | Green | `0xFF4CAF50` | Employee had perfect attendance (punch-in at exactly scheduled time and proper punch-out) |
| Manual Adjustments | Green | `0xFF4CAF50` | Timesheet contains manual adjustments |
| Default Workday | Light Red | `0xFFE57373` | Default color for workdays when no other conditions apply |

### Logic Flow

1. First, the function checks the `label` parameter:
   - If it's 'Holiday', returns blue.
   - If it's 'Offday', returns grey.
   - If it's 'Workday', proceeds to further conditions.

2. For workdays, it checks several conditions in the following order:
   - Long-term absence: If title contains '[LR] Absent', returns red.
   - Regular absence: If title contains 'Absent', returns orange.
   - Missing punch records: If punch-in or punch-out is null AND title contains 'Missing' or 'N/A', returns red.
   - Leave: If leave details are present, returns purple.
   - Early departure: If punch-out delta starts with '-1', returns light red.
   - Late arrival: If punch-in delta starts with '-', returns green.
   - Perfect attendance: If punch-in delta is exactly '00:00:00' and punch-out delta doesn't start with '-1', returns green.
   - Manual adjustments: If adjustments are present, returns green.
   - Default: Returns light red if none of the above conditions are met.

3. For any other label value, returns null. Currently, all the conditions are handled, so this null condition does not occur.

### Example Usage

```dart
// Example usage in a timesheet cell rendering function
Widget buildTimeSheetCell(String label, TimeSheetResult data) {
  return Container(
    color: getCellColor(label: label, timeSheet: data),
    child: Text(data.title),
  );
}
```

### 🚨 Notes
- The color coding provides visual cues for quickly identifying attendance patterns and issues.
- The priority of conditions matters - the function returns the first matching color in the sequence.
- punchindelta -1 is considered as early in, punchoutdelta -1 is considered as early out
