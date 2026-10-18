# Runner's Progress Tracker

A lightweight, browser-based PHP application for calculating and recording a runner's marathon progress. It compares the runner's current pace with the speed required to finish the remaining distance within a target time.

The application stores the current browser session's history in PHP session data and appends every valid calculation to a plain-text CSV-style file named `race_data.txt`.

## Features

- Record a runner's name.
- Enter total race distance in kilometers.
- Enter distance covered in kilometers.
- Enter elapsed time in hours.
- Enter target completion time in hours.
- Calculate current speed in km/h.
- Calculate the required remaining speed in km/h.
- Detect when no target time remains.
- Display historical race records in reverse chronological order.
- Highlight the newest record.
- Delete records from the current PHP session.
- Append valid records to `race_data.txt`.
- Use a responsive layout that switches to one form column on small screens.
- Use a configurable background image at `bg_img.jpg`.

## How Calculations Work

### Current speed

```text
current speed = distance covered / elapsed time
```

The result is formatted to two decimal places and measured in km/h.

### Required speed

```text
remaining distance = total distance - distance covered
remaining time = target time - elapsed time
required speed = remaining distance / remaining time
```

If the remaining time is zero or negative, the application displays:

```text
No time left. You already lost the race.
```

The required-speed message is stored in the record instead of a numeric value in that case.

## Requirements

- PHP 7.1 or later; PHP 8.x is recommended.
- PHP sessions enabled.
- A PHP-compatible web server.
- A writable directory for `race_data.txt`.
- Optional: `bg_img.jpg` in the directory served for the page.

No database server or PHP database extension is required.

## Installation

1. Place `RunnerProgressTracker.php` in a PHP-accessible web directory.
2. Ensure the PHP process can create and append to `race_data.txt`.
3. Optionally place a background image named `bg_img.jpg` in the same web directory.
4. Start a PHP web server.
5. Open the PHP file in a browser.

### Run with PHP's built-in server

From the directory containing the PHP file:

```bash
php -S localhost:8000
```

Open:

```text
http://localhost:8000/RunnerProgressTracker.php
```

Do not open the file with a `file://` URL; PHP must process it through a web server.

## Using the Application

1. Enter the runner's name.
2. Enter the total marathon distance in kilometers.
3. Enter the distance covered so far in kilometers.
4. Enter elapsed time in hours.
5. Enter the target completion time in hours.
6. Select **Calculate**.

The result is added to **Historical Race Data**. The newest row appears at the top and is highlighted.

The table displays:

- Runner
- Total distance
- Distance covered
- Elapsed time
- Target time
- Current speed
- Required speed
- Delete action

## Validation Rules

A submission is accepted only when:

- The runner name is not empty.
- Total distance is numeric and greater than zero.
- Distance covered is numeric and zero or greater.
- Elapsed time is numeric and greater than zero.
- Target time is numeric and greater than zero.
- Distance covered does not exceed total distance.

Invalid submissions display an error message and are not saved.

The current implementation accepts numeric values such as decimals, but it does not enforce a separate maximum for distance, elapsed time, or target time.

## Storage

### PHP session

The application initializes:

```php
$_SESSION['race_data']
```

Each valid calculation is stored as an array in that session. The visible historical table is built from this session array.

The session is browser/session-specific and may disappear when the session expires, cookies are cleared, or the PHP session storage is reset.

### `race_data.txt`

Each valid calculation is appended to:

```text
race_data.txt
```

The saved fields are written as one comma-separated line:

```text
name,total,covered,elapsed,target,current_speed,required_speed
```

The file has no header row. The path is passed as the relative filename `race_data.txt`, so the effective location depends on the PHP process's working directory and server configuration. Verify the location in deployment rather than assuming it is always next to the script.

The text file is append-only in the current application. Deleting a row from the web page removes it from the current session only; it does not remove the corresponding line from `race_data.txt`.

## Delete Behavior

Select **Delete** beside a history row and confirm the browser prompt.

The application removes the selected entry from `$_SESSION['race_data']` and redirects back to the PHP page to avoid duplicate form submission.

The external `race_data.txt` file is not rewritten. Therefore, the file can contain records that no longer appear in the current session table.

## Request Behavior

The application does not define named REST routes or JSON responses. It uses one PHP page and ordinary POST forms.

### Calculate request

```http
POST /RunnerProgressTracker.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=Alex&total=42.195&covered=20&elapsed=2&target=4
```

The actual form fields are:

| Field | Meaning |
|---|---|
| `name` | Runner's name |
| `total` | Total race distance in km |
| `covered` | Distance covered in km |
| `elapsed` | Time elapsed in hours |
| `target` | Target completion time in hours |

### Delete request

```http
POST /RunnerProgressTracker.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded

delete_index=0
```

`delete_index` is the zero-based index in the original session array, not the displayed reversed-row position.

## Project Structure

```text
RunnerProgressTracker.php        Application, calculations, forms, and display
race_data.txt                    Append-only calculation history created at runtime
bg_img.jpg                       Optional background image
README_RunnerProgressTracker.md  Project README and technical documentation
RUNNER_PROGRESS_USER_MANUAL.md   User manual and form-action reference
```

## Troubleshooting

### Records disappear after reopening the browser

The visible table is stored in the PHP session, not loaded from `race_data.txt`. Session data can expire or be cleared. The file may still contain earlier records.

### `race_data.txt` is not created

Check that the PHP process has permission to write in its working directory. Also check the web-server process's current working directory because the code uses a relative filename.

### The background image is missing

Place `bg_img.jpg` in the same directory served for the page, or update the CSS background URL.

### Required speed shows a message instead of a number

The target time has already been reached or passed. Enter a target time greater than the elapsed time to receive a numeric required-speed result.

### The page does not run

Serve the file through PHP rather than opening it directly from the filesystem. Confirm that PHP sessions are enabled.

## Security and Production Considerations

Before public deployment, consider:

- CSRF protection for calculation and delete forms.
- Server-side validation with explicit numeric conversion and sensible upper bounds.
- Authentication if runners' names or progress data are private.
- A database or structured storage format instead of an unescaped CSV-style file.
- Safe file locking with `LOCK_EX` when appending records.
- A strategy to synchronize deletes between the session and persistent storage.
- A fixed absolute storage path outside the public web root.
- Restricting direct downloads of `race_data.txt` if it contains personal information.
- Escaping all rendered user-provided values; the current history output is escaped, but error output should also be escaped if it becomes dynamic.
- Use of HTTPS in production.

## License

No license is currently specified. Add a `LICENSE` file before distributing or deploying the application if a specific license is intended.
