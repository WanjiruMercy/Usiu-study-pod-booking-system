**Project Overview**

This is a single-page JavaScript application for managing study pod bookings at USIU-Africa library. Librarians can use this system to:

Register new bookings in real-time

Enforce university booking rules

View operational insights

Manage existing bookings

**Features Implemented Include:**

**Booking Form:**
Dynamic pod selection dropdown populated from pods array

Time input restricted to 1-hour blocks between 08:00-20:00

Comma-separated student IDs input with automatic parsing

Comprehensive validation with clear error messages

**Bookings Table:**

Real-time display of all bookings (initial + new)

Each booking shows:Sequential number, pod ID, time slot, number of students, list of student IDs and the remove button

Event delegation for remove functionality

**Insights Panel includes:**

Total bookings made today

Total unique students served

Busiest hour (most students across all pods)

Fill-rate per pod (percentage of capacity used)

Flagged duplicate attempts count

**Utility Functions:**

parseStudentIds() - Processes comma-separated input

isWithinOperatingHours() - Validates time slots

findBooking() - Locates existing bookings

hasCrossPodClash() - Checks for student time conflicts

recomputeInsights() - Updates all statistics

**Code Structure:**

// Pod definitions (capacity 4 students each)
const pods = [
  { id: "POD-A", capacity: 4 },
  { id: "POD-B", capacity: 4 },
  { id: "POD-C", capacity: 4 }
];

// Initial bookings
let bookings = [
  { podId: "POD-A", time: "09:00", students: ["SIT-001", "SIT-045"] },
  { podId: "POD-B", time: "10:00", students: ["SMC-210"] }
];

// Track duplicate booking attempts
let duplicateAttempts = 0;
Initialization:
function init() {
  populatePodSelect();  // Fill pod dropdown
  renderBookingsTable(); // Display initial bookings
  recomputeInsights();  // Calculate initial stats

  // Set up event listeners
  bookingForm.addEventListener('submit', handleFormSubmit);
  bookingsTable.addEventListener('click', handleTableClick);
}

**Validation Rules**

**Capacity Rule:**

Checks if adding **new students would exceed pod capacity** and **considers both new bookings and additions to existing bookings**

// In validateBooking() function
const pod = pods.find(p => p.id === podId);
const existingBooking = findBooking(podId, time);
const totalStudents = existingBooking 
  ? existingBooking.students.length + studentIds.length 
  : studentIds.length;

if (totalStudents > pod.capacity) {
  showError(`This pod can only accommodate ${pod.capacity} students at this time`);
  return false;
}

**Duplicate Booking Rule:**
Prevents same student being booked twice in same pod/time and Increments duplicate attempts counter

if (existingBooking) {
  const duplicates = studentIds.filter(id => 
    existingBooking.students.includes(id)
  );
  if (duplicates.length > 0) {
    duplicateAttempts++;
    showError(`Students ${duplicates.join(', ')} already booked in this pod at this time`);
    return false;
  }
}

**Cross-Pod Clash Rule:**
Uses **Array.some()** to check for conflicts and returns true if student is booked in another pod at same time

function hasCrossPodClash(studentId, timeString, currentPodId) {
  return bookings.some(booking => 
    booking.time === timeString && 
    booking.podId !== currentPodId && 
    booking.students.includes(studentId)
  );
}

**Operating Hours Rule:**
Uses **strict comparisons (===) ** to prevent type coercion and also ensures 08:00 is valid, 20:00 is invalid (exclusive end)

function isWithinOperatingHours(timeString) {
  const time = timeString.split(':');
  const hours = parseInt(time[0], 10);
  const minutes = parseInt(time[1], 10);
  
  // 08:00 (inclusive) to 20:00 (exclusive)
  return (hours > 8 || (hours === 8 && minutes === 0)) && 
         (hours < 20);
}

**Strict Equality Justification:**
Using** === **ensures both **type and value equality**, **prevents unexpected behavior with different data types** and is especially **important for time comparisons ("09:00" vs new String("09:00"))**

// Example from findBooking()
return bookings.find(booking => 
  booking.podId === podId && booking.time === timeString
);

**DOM Manipulation:**

*Dynamic Select Population*:
**Clears existing options, creates new option elements from pods array and uses textContent for safe text injection**

function populatePodSelect() 
{
  podSelect.innerHTML = '';
  pods.forEach(pod => {
    const option = document.createElement('option');
    option.value = pod.id;
    option.textContent = pod.id;
    podSelect.appendChild(option);
  });
  
}

**Table Rendering:**
Uses plain DOM methods (createElement, appendChild), manual for loop instead of array helpers like map and data attributes for remove button indexing.

function renderBookingsTable() {
  bookingsTable.innerHTML = '';
  
  for (let i = 0; i < bookings.length; i++) {
    const booking = bookings[i];
    const row = document.createElement('tr');
    
    // Create and append all table cells
    // ...
    
    bookingsTable.appendChild(row);
  }
}

**Insights Calculation:**
Unique Students Count:
The lines below aid in manual implementation without using Set and use of nested loops to collect all unique student IDs

function countUniqueStudents() {
  const allStudents = [];
  bookings.forEach(booking => {
    booking.students.forEach(student => {
      if (!allStudents.includes(student)) {
        allStudents.push(student);
      }
    });
  });
  return allStudents.length;
}

**Busiest Hour Calculation:**
This pre-initializes all possible time slots, aggregates student counts per hour and finds maximum with manual comparison

function findBusiestHour() {
  const hourCounts = {};
  
  // Initialize counts for all possible hours
  for (let hour = 8; hour < 20; hour++) {
    hourCounts[`${hour.toString().padStart(2, '0')}:00`] = 0;
  }
  // Count students per hour
  bookings.forEach(booking => {
    hourCounts[booking.time] += booking.students.length;
  });
  // Find hour with max count
  let maxHour = null;
  let maxCount = 0;
  for (const hour in hourCounts) {
    if (hourCounts[hour] > maxCount) {
      maxCount = hourCounts[hour];
      maxHour = hour;
    }
  }
  return maxCount > 0 ? `${maxHour} (${maxCount} students)` : null;
}

**Pod Fill Rate:**
Calculates percentage of used capacity, and handles pods with no bookings

function calculatePodFillRate(podId) {
  const pod = pods.find(p => p.id === podId);
  const podBookings = bookings.filter(b => b.podId === podId);
  
  if (podBookings.length === 0) return 0;
  const totalBookedSeats = podBookings.reduce((sum, booking) => {
    return sum + booking.students.length;
  }, 0);
  const totalPossibleSeats = pod.capacity * podBookings.length;
  const fillRate = (totalBookedSeats / totalPossibleSeats) * 100;
  return roundToOneDecimal(fillRate);
}

**Event Handling:**
**Form Submission** 
Prevents default form submission, clears previous errors, and validates before adding booking

function handleFormSubmit(e) {
  e.preventDefault();
  clearErrors();
  
  const podId = podSelect.value;
  const time = timeInput.value;
  const studentIds = parseStudentIds(studentsInput.value);
  
  if (validateBooking(podId, time, studentIds)) {
    addBooking(podId, time, studentIds);
    renderBookingsTable();
    recomputeInsights();
    studentsInput.value = '';
    studentsInput.focus();
  }
}

**On event delegation: Identifies bookings and updates UI after removal**

function handleTableClick(e) {
  if (e.target.tagName === 'BUTTON') {
    const index = parseInt(e.target.dataset.index, 10);
    bookings.splice(index, 1);
    renderBookingsTable();
    recomputeInsights();
  }
}

**Student ID Parsing:**
Converts to **uppercase for consistency,** and **removes entires from malformed input**

function parseStudentIds(inputString) {
  return inputString.split(',')
    .map(id => id.trim().toUpperCase()) // Normalize to uppercase
    .filter(id => id !== ''); // Remove empty entries
}

**Time Validation:**
Handles cases such as: **Empty time, invalid format** and **times outside the standard 08:00-20:00 range**

// In isWithinOperatingHours()
const time = timeString.split(':');
const hours = parseInt(time[0], 10);
const minutes = parseInt(time[1], 10);

**Empty Student List:**
This handles situations such as: **Empty input, only commas** and **only whitespace**

// In validateBooking()
if (studentIds.length === 0) {
  showError("Please enter at least one valid student ID");
  return false;
}
