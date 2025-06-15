# Handling and Manipulating Strings in Micro800 Controllers

## Introduction

String data is essential in automation for tasks such as operator messages, serial communication, data logging, and recipe management. In Micro800 controllers, string manipulation is accomplished using a combination of built-in instructions, function blocks, and Structured Text (ST) operations. This document explores the available instructions, their parameters, and practical usage patterns for robust string handling.

---

## 1. String Data Types in Micro800

- **STRING[n]**: A variable-length string with a maximum of `n` characters.
- **BYTE/USINT Arrays**: Sometimes used to represent or buffer ASCII data for communication.

---

## 2. Core String Instructions and Examples

### 2.1. COP (Copy)
- **Purpose**: Copies the contents of one string (or array) to another.
- **Usage**:
    - Basic: `COP(Source, Dest, Length);`
    - With offsets: `COP(Source, SourceOffset, Dest, DestOffset, Length);`
    - With byte swap (if supported): `COP(Source, SourceOffset, Dest, DestOffset, Length, SwapBytes);`
- **Parameters:**
    - `Source`: The source string or array.
    - `SourceOffset`: Starting index in the source (0-based).
    - `Dest`: The destination string or array.
    - `DestOffset`: Starting index in the destination (0-based).
    - `Length`: Number of elements (bytes or words) to copy.
    - `SwapBytes` (optional): If supported, enables byte swapping during the copy (useful for communication with devices using different endianness).

**Example: Basic Copy**
Copies the entire contents of one string to another.
```st
VAR
    str1 : STRING[10] := 'Hello';
    str2 : STRING[10] := '';
END_VAR
```

```st
COP(str1, str2, 1); // str2 now contains 'Hello'
```

**Example: Partial Copy with Offsets**
Copies a portion of the source string (starting at index 2) to the destination (starting at index 0), for 3 characters.
```st
VAR
    src : STRING[10] := 'ABCDEFGHIJ';
    dest : STRING[10] := '';
END_VAR
```

```st
COP(src, 2, dest, 0, 3); // dest = 'CDE'
```

**Example: Byte Swapping**
Copies 4 bytes from the source to the destination, swapping the byte order (if supported by the platform).
```st
VAR
    src : STRING[10] := 'ABCD';
    dest : STRING[10] := '';
END_VAR
```

```st
COP(src, 0, dest, 0, 4, TRUE);
```

> Note: The availability of source/destination offsets and byte swapping depends on your controller firmware and programming environment. Consult the official documentation for your platform for exact syntax and support.

---

### 2.2. INSERT (Insert String)
- **Purpose**: Inserts a substring at a user-defined position within another string.
- **Usage**: `Result := INSERT(IN, Str, Pos);`
- **Parameters:**
    - `IN`: Initial string.
    - `Str`: String to be inserted.
    - `Pos`: Position for insertion (insertion is before this position; first valid position is 1).
- **Result:**
    - Returns a modified string with `Str` inserted into `IN` at the specified position.
    - If `Pos <= 0`, the result is an empty string.
    - If `Pos` is greater than the length of `IN`, the result is the concatenation of `IN` and `Str`.
- **Supported Languages:** Function Block Diagram, Ladder Diagram, Structured Text.
- **Supported Controllers:** Micro810, Micro820, Micro830, Micro850, Micro870, Micro800 Simulator.
- **Reference:** Official documentation and examples: `109066.htm` (INSERT instruction).

**Example: Insert in the Middle**
Inserts the string 'Frank ' into 'Mr JONES' at position 4, resulting in 'Mr Frank JONES'.
```st
VAR
    MyName : STRING[20];
END_VAR
```

```st
MyName := INSERT('Mr JONES', 'Frank ', 4); // 'Mr Frank JONES'
```

**Example: Append Using MLEN**
Appends ' World' to 'Hello' by calculating the position after the last character using MLEN.
```st
VAR
    base : STRING[20] := 'Hello';
    insertStr : STRING[10] := ' World';
    result : STRING[30];
    pos : DINT;
END_VAR
```

```st
pos := MLEN(base) + 1;
result := INSERT(base, insertStr, pos); // result = 'Hello World'
```

**Example: Insert at Specific Position**
Inserts the name 'Frank' into the greeting at position 6, resulting in 'Dear Frank, welcome!'.
```st
VAR
    greeting : STRING[30] := 'Dear , welcome!';
    name : STRING[10] := 'Frank';
    pos : DINT;
END_VAR
```

```st
pos := 6;
greeting := INSERT(greeting, name, pos); // greeting = 'Dear Frank, welcome!'
```

---

### 2.3. MLEN (String Length)
- **Purpose**: Calculates the length of a string.
- **Usage**: `Result := MLEN(Str);`
- **Result**: Returns the number of characters in the input string.
- **Supported Languages**: Function Block Diagram, Ladder Diagram, Structured Text.
- **Reference:** See `109089.htm` (official documentation).

**Example:**
Calculates the length of the string 'Micro800' and stores it in the variable 'length'.
```st
VAR
    myString : STRING[20] := 'Micro800';
    length : DINT;
END_VAR
```

```st
length := MLEN(myString); // length = 8
```

---

### 2.4. FIND, LEFT, MID, RIGHT (Substring and Search)
- **Purpose**: Locate delimiters and extract substrings.
- **Usage:**
    - `pos := FIND(string, delimiter);`
    - `leftPart := LEFT(string, count);`
    - `midPart := MID(string, start, count);`
    - `rightPart := RIGHT(string, count);`

**Example: Split at Delimiter**
Splits the string 'TEMP:23.5' at the colon, extracting 'TEMP' as the key and '23.5' as the value.
```st
VAR
    input : STRING[30] := 'TEMP:23.5';
    delimiter : STRING[2] := ':';
    pos : DINT;
    key : STRING[10];
    value : STRING[10];
END_VAR
```

```st
pos := FIND(input, delimiter);
IF pos > 0 THEN
    key := LEFT(input, pos - 1);
    value := RIGHT(input, MLEN(input) - pos);
END_IF;
```

**Example: Extract Between Delimiters**
Extracts the substring between two '|' characters in 'A|B|C', resulting in 'B'.
```st
VAR
    input : STRING[30] := 'A|B|C';
    delimiter1 : STRING[2] := '|';
    delimiter2 : STRING[2] := '|';
    pos1, pos2 : DINT;
    middle : STRING[10];
END_VAR
```

```st
pos1 := FIND(input, delimiter1);
pos2 := FIND(MID(input, pos1 + 1, MLEN(input) - pos1), delimiter2);
IF pos1 > 0 AND pos2 > 0 THEN
    middle := MID(input, pos1 + 1, pos2 - 1);
END_IF;
```

---

## 3. Advanced String Manipulation Scenarios

### 3.1. Copying Between STRING and BYTE Arrays
- Use `COP` to convert between string and byte array representations for communication or parsing.

**Example:**
Converts a string to a byte array for communication or parsing.
```st
VAR
    str : STRING[8] := 'DATA';
    bytes : ARRAY[0..7] OF BYTE;
END_VAR
```

```st
COP(str, bytes, 1); // bytes now contains ASCII codes for 'D','A','T','A'
```

### 3.2. Dynamic String Assembly
- Use `INSERT` to build messages dynamically by inserting substrings at desired positions.
- Use `MLEN` to calculate the position for insertion, such as appending to the end of a string or inserting at a specific offset.

**Example: Appending a value to a base string**
Appends the value string to the base string using INSERT and MLEN.
```st
VAR
    base : STRING[20] := 'Temp: ';
    value : STRING[5] := '23.5';
    msg : STRING[25];
    pos : DINT;
END_VAR
```

```st
pos := MLEN(base) + 1;
msg := INSERT(base, value, pos); // msg = 'Temp: 23.5'
```

**Example: Inserting a substring in the middle of another string**
Inserts a substring into the middle of another string by calculating the position.
```st
VAR
    original : STRING[20] := 'Micro800';
    insertStr : STRING[5] := '-PLC-';
    result : STRING[30];
    pos : DINT;
END_VAR
```

```st
pos := 6;
result := INSERT(original, insertStr, pos); // result = 'Micro-PLC-800'
```

**Example: Building a message with multiple dynamic parts**
Builds a message by inserting multiple parts in sequence.
```st
VAR
    msg : STRING[40] := '';
    part1 : STRING[10] := 'ID:';
    id : STRING[5] := '1234';
    part2 : STRING[10] := ';VAL:';
    val : STRING[5] := '56';
    pos : DINT;
END_VAR
```

```st
msg := INSERT(msg, part1, 1);
pos := MLEN(msg) + 1;
msg := INSERT(msg, id, pos);
pos := MLEN(msg) + 1;
msg := INSERT(msg, part2, pos);
pos := MLEN(msg) + 1;
msg := INSERT(msg, val, pos); // msg = 'ID:1234;VAL:56'
```

---

### 3.3. Parsing JSON Strings

#### Parsing a Simple JSON String
Extracts the 'id', 'name', and 'status' fields from a JSON string using FIND, MID, and MLEN.
```st
VAR
    json : STRING[80] := '{"id":123,"name":"Pump1","status":"ON"}';
    idStr : STRING[10];
    name : STRING[20];
    status : STRING[10];
    pos_id : DINT;
    pos_name : DINT;
    pos_status : DINT;
    pos_colon : DINT;
    pos_comma : DINT;
    pos_quote1 : DINT;
    pos_quote2 : DINT;
    temp : STRING[40];
END_VAR
```

```st
// Extract 'id' value
pos_id := FIND(json, '"id"');
IF pos_id > 0 THEN
    pos_colon := FIND(MID(json, pos_id, MLEN(json) - pos_id + 1), ':');
    IF pos_colon > 0 THEN
        pos_comma := FIND(MID(json, pos_id + pos_colon, MLEN(json) - (pos_id + pos_colon) + 1), ',');
        IF pos_comma > 0 THEN
            idStr := MID(json, pos_id + pos_colon, pos_comma - 1);
        ELSE
            idStr := MID(json, pos_id + pos_colon, MLEN(json) - (pos_id + pos_colon) - 1);
        END_IF;
    END_IF;
END_IF;

// Extract 'name' value
pos_name := FIND(json, '"name"');
IF pos_name > 0 THEN
    pos_quote1 := FIND(MID(json, pos_name, MLEN(json) - pos_name + 1), '"');
    pos_quote1 := pos_quote1 + pos_name;
    pos_quote2 := FIND(MID(json, pos_quote1 + 1, MLEN(json) - pos_quote1), '"');
    IF pos_quote2 > 0 THEN
        name := MID(json, pos_quote1 + 1, pos_quote2 - 1);
    END_IF;
END_IF;

// Extract 'status' value
pos_status := FIND(json, '"status"');
IF pos_status > 0 THEN
    pos_quote1 := FIND(MID(json, pos_status, MLEN(json) - pos_status + 1), '"');
    pos_quote1 := pos_quote1 + pos_status;
    pos_quote2 := FIND(MID(json, pos_quote1 + 1, MLEN(json) - pos_quote1), '"');
    IF pos_quote2 > 0 THEN
        status := MID(json, pos_quote1 + 1, pos_quote2 - 1);
    END_IF;
END_IF;
```

#### Parsing a JSON Array
Extracts each value from a JSON array string and stores them in a string array.
```st
VAR
    json : STRING[40] := '{"values":[10,20,30]}';
    arrayStr : STRING[30];
    value : STRING[5];
    values : ARRAY[1..10] OF STRING[5];
    pos_start : DINT;
    pos_end : DINT;
    pos_comma : DINT;
    idx : DINT := 1;
    len : DINT;
END_VAR
```

```st
pos_start := FIND(json, '[');
pos_end := FIND(json, ']');
IF pos_start > 0 AND pos_end > pos_start THEN
    arrayStr := MID(json, pos_start + 1, pos_end - pos_start - 1);
    len := MLEN(arrayStr);
    WHILE len > 0 DO
        pos_comma := FIND(arrayStr, ',');
        IF pos_comma > 0 THEN
            value := LEFT(arrayStr, pos_comma - 1);
            arrayStr := RIGHT(arrayStr, len - pos_comma);
            len := MLEN(arrayStr);
        ELSE
            value := arrayStr;
            len := 0;
        END_IF;
        values[idx] := value;
        idx := idx + 1;
    END_WHILE;
END_IF;
```

#### Converting a JSON String Value to a Boolean Variable
Extracts a string value from JSON and converts 'ON' or 'TRUE' to a BOOL variable.
```st
VAR
    json : STRING[30] := '{"status":"ON"}';
    statusStr : STRING[10];
    pos_status : DINT;
    pos_quote1 : DINT;
    pos_quote2 : DINT;
    statusBool : BOOL := FALSE;
END_VAR
```

```st
pos_status := FIND(json, '"status"');
IF pos_status > 0 THEN
    pos_quote1 := FIND(MID(json, pos_status, MLEN(json) - pos_status + 1), '"');
    pos_quote1 := pos_quote1 + pos_status;
    pos_quote2 := FIND(MID(json, pos_quote1 + 1, MLEN(json) - pos_quote1), '"');
    IF pos_quote2 > 0 THEN
        statusStr := MID(json, pos_quote1 + 1, pos_quote2 - 1);
    END_IF;
END_IF;
IF (statusStr = 'ON') OR (statusStr = 'TRUE') THEN
    statusBool := TRUE;
ELSE
    statusBool := FALSE;
END_IF;
```

#### Building a Properly Formatted JSON String with an Array
Builds a JSON string with an array field by assembling the string step by step.
```st
VAR
    json : STRING[100] := '{';
    id : STRING[5] := '123';
    name : STRING[10] := 'Pump1';
    status : STRING[5] := 'ON';
    values : ARRAY[1..3] OF STRING[5] := ('10', '20', '30');
    arrStr : STRING[30] := '';
    i : DINT;
    pos : DINT;
END_VAR
```

```st
FOR i := 1 TO 3 DO
    arrStr := INSERT(arrStr, values[i], MLEN(arrStr) + 1);
    IF i < 3 THEN
        arrStr := INSERT(arrStr, ',', MLEN(arrStr) + 1);
    END_IF;
END_FOR;
json := INSERT(json, '"id":', MLEN(json) + 1);
json := INSERT(json, id, MLEN(json) + 1);
json := INSERT(json, ',', MLEN(json) + 1);
json := INSERT(json, '"name":"', MLEN(json) + 1);
json := INSERT(json, name, MLEN(json) + 1);
json := INSERT(json, '"', MLEN(json) + 1);
json := INSERT(json, ',', MLEN(json) + 1);
json := INSERT(json, '"status":"', MLEN(json) + 1);
json := INSERT(json, status, MLEN(json) + 1);
json := INSERT(json, '"', MLEN(json) + 1);
json := INSERT(json, ',', MLEN(json) + 1);
json := INSERT(json, '"values":[', MLEN(json) + 1);
json := INSERT(json, arrStr, MLEN(json) + 1);
json := INSERT(json, ']}', MLEN(json) + 1);
```

---

## 4. Best Practices
- Always check string lengths before concatenation or copy to avoid truncation.
- Use offsets and lengths in `COP` for partial string operations.
- For serial communication, monitor status and error outputs from AWA/ARD/ARL.
- Use structured text for complex parsing and assembly, and ladder for simple moves and checks.

---

## 5. Troubleshooting
- If a string is truncated, check the destination size and the length parameter.
- If serial communication fails, check the channel, buffer, and error/status outputs.
- For unexpected results in substring operations, verify start positions and counts.

---

## Appendix: Example – JSONMaker_V2 for MQTT/IoT

### Source Code: JSONMaker_V2.st

**Purpose:**
This function block creates a JSON-formatted string from up to 20 input strings, suitable for sending to an MQTT server as part of an IoT solution.

**Inputs:**
- `InArray`: Array of up to 20 string values to be included as JSON values.
- `InputNames`: Optional array of up to 20 string keys for the JSON object. If a key is blank, a fallback letter (A-T) is used.

**Outputs:**
- `JsonString`: The resulting JSON string, e.g. `{ "A":"val1", "B":"val2" }`.
- `FBENO`: Indicates completion of the function block operation.

**Operation:**
1. The function block is enabled by setting `FBEN` to TRUE.
2. The output string is initialized with an opening brace `{`.
3. The code loops through all 20 possible entries in `InArray`.
4. For each non-empty entry:
   - If it is the first entry, no comma is added. For subsequent entries, a comma is inserted.
   - The key is determined: if `InputNames[i]` is not empty, it is used; otherwise, a fallback letter (A-T) is generated using `CHAR(65 + i)`.
   - The key and value are inserted in JSON format: `"key":"value"`.
5. After all entries are processed, a closing brace `}` is appended.
6. The done flag `FBENO` is set to TRUE.
7. If the function block is not enabled, `FBENO` is set to FALSE.

**Key Features:**
- Only non-empty input values are included in the output JSON.
- Supports custom key names or automatic fallback keys.
- Handles comma placement for valid JSON formatting.
- Output is limited to 255 characters (STRING[255]).

**Usage Notes:**
- Designed for use in Rockwell Micro800 Structured Text.
- Useful for preparing JSON payloads for MQTT or other IoT protocols.
- Can be adapted for different array sizes or key naming conventions as needed.

```st
VAR
   InArray : ARRAY[0..19] OF STRING[32];
   InputNames : ARRAY[0..19] OF STRING[32];
   JsonString : STRING[255];
   i      : DINT;
   first  : BOOL;
   strLen : DINT;
   key    : STRING[32];
   FBEN  : BOOL;
   FBENO : BOOL;
END_VAR
```
```st
IF FBEN THEN
    first := TRUE;
    JsonString := '{';
    FOR i := 0 TO 19 DO
        IF MLEN(InArray[i]) > 0 THEN
            IF first THEN
                first := FALSE;
            ELSE
                strLen := MLEN(JsonString);
                JsonString := INSERT(JsonString, ',', strLen + 1);
            END_IF;
            IF MLEN(InputNames[i]) > 0 THEN
                key := InputNames[i];
            ELSE
                key := CHAR(65 + i);
            END_IF;
            strLen := MLEN(JsonString);
            JsonString := INSERT(JsonString, '"', strLen + 1);
            strLen := MLEN(JsonString);
            JsonString := INSERT(JsonString, key, strLen + 1);
            strLen := MLEN(JsonString);
            JsonString := INSERT(JsonString, '":"', strLen + 1);
            strLen := MLEN(JsonString);
            JsonString := INSERT(JsonString, InArray[i], strLen + 1);
            strLen := MLEN(JsonString);
            JsonString := INSERT(JsonString, '"', strLen + 1);
        END_IF;
    END_FOR;
    strLen := MLEN(JsonString);
    JsonString := INSERT(JsonString, '}', strLen + 1);
    FBENO := TRUE;
ELSE
    FBENO := FALSE;
END_IF;
```

---

## Appendix: Example – BuildTimeStampUDFB (Unique Timestamp Generator)

### Source Code: BuildTimeStampUDFB.st

**Purpose:**
Generates a unique, compact timestamp string for use in data logging, traceability, or as a unique identifier in IoT applications.

**Inputs:**
- `FBEN`: Enable input for the function block.

**Outputs:**
- `FBENO`: Indicates completion of the function block operation.
- `TimestampStr`: The resulting timestamp string in the format YYMMDDSSSSS.

**Operation:**
1. When enabled (`FBEN` is TRUE), the function block reads the current date and time from the PLC's RTC using the `RTCREAD` function block.
2. It calculates the number of seconds since midnight by converting hours, minutes, and seconds to a total count.
3. The year, month, and day are converted to two-digit, zero-padded strings. The seconds since midnight are converted to a five-digit, zero-padded string.
4. These components are concatenated in the order YYMMDDSSSSS using the `INSERT` instruction.
5. The output string is available in `TimestampStr` and the done flag `FBENO` is set to TRUE.
6. If not enabled, `FBENO` is set to FALSE.

**Key Features:**
- Produces a compact, sortable timestamp string suitable for file names, record IDs, or event tags.
- Ensures all components are zero-padded for consistent length.
- Uses only standard Micro800 Structured Text and instructions.

**Usage Notes:**
- Requires the `RTCREAD` function block and a real-time clock in the controller.
- Can be adapted for different timestamp formats as needed.

```st
VAR
    FBEN : BOOL;
    FBENO : BOOL;
    TimestampStr : STRING[12];
    stRTC : RTC := (Year := 0, Month := 0, Day := 0, Hours := 0, Minutes := 0, Seconds := 0);
    SecondsSinceMidnight : DINT;
    fbRTCRead : RTCREAD;
    sYear : STRING[3];
    sMonth : STRING[3];
    sDay : STRING[3];
    sSeconds : STRING[6];
END_VAR
```
```st
IF FBEN THEN
    fbRTCRead(TRUE);
    stRTC := fbRTCRead.RTCData;
    SecondsSinceMidnight := ANY_TO_DINT(stRTC.Hours) * 3600
                           + ANY_TO_DINT(stRTC.Minutes) * 60
                           + ANY_TO_DINT(stRTC.Seconds);
    sYear := ANY_TO_STRING(ANY_TO_DINT(stRTC.Year) - 2000);
    IF MLEN(sYear) < 2 THEN
        sYear := INSERT('0', sYear, 1);
    END_IF;
    sMonth := ANY_TO_STRING(ANY_TO_DINT(stRTC.Month));
    IF MLEN(sMonth) < 2 THEN
        sMonth := INSERT(sMonth, '0', 1);
    END_IF;
    sDay := ANY_TO_STRING(ANY_TO_DINT(stRTC.Day));
    IF MLEN(sDay) < 2 THEN
        sDay := INSERT(sDay, '0', 1);
    END_IF;
    sSeconds := ANY_TO_STRING(SecondsSinceMidnight);
    CASE MLEN(sSeconds) OF
        1: sSeconds := INSERT(sSeconds, '0000', 1);
        2: sSeconds := INSERT(sSeconds, '000', 1);
        3: sSeconds := INSERT(sSeconds, '00', 1);
        4: sSeconds := INSERT(sSeconds, '0', 1);
    END_CASE;
    TimestampStr := '';
    TimestampStr := INSERT(sYear, TimestampStr, 1);
    TimestampStr := INSERT(sMonth, TimestampStr, 1);
    TimestampStr := INSERT(sDay, TimestampStr, 1);
    TimestampStr := INSERT(sSeconds, TimestampStr, 1);
    FBENO := TRUE;
ELSE
    FBENO := FALSE;
END_IF;
```
