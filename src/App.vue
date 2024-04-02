<script>
  export default {
    name: 'CustomTable', 
  data() {
    return {
      students: [
        { id: 's1', name: 'Alice' },
        { id: 's2', name: 'Bob' },
        { id: 's3', name: 'Charlie' },
      ],
      subjects: [
        { id: 'math', name: 'Math' },
        { id: 'science', name: 'Science' },
        { id: 'history', name: 'History' },
      ],
      marks: [
        // Each sub-array represents marks for a subject in the order of students
        [85, 75, 95], // Math
        [80, 82, 78], // Science
        [90, 88, 92], // History
      ],
      currentSort: {
        type: null, // 'subject' or 'student'
        id: null,
        direction: 'ascending', // or 'descending'
      },

      isEditMode: false, 
      editableCellId: null, // Tracks the currently editable cell by combining studentId and subjectId
    };
    
  },
  methods: {
    toggleEditMode() {
    this.isEditMode = !this.isEditMode;
  },

  updateMark(subjectId, studentId, event) {
  if (!this.isEditMode) return;
  const subjectIndex = this.subjects.findIndex(subject => subject.id === subjectId);
  const studentIndex = this.students.findIndex(student => student.id === studentId);
  this.marks[subjectIndex][studentIndex] = parseInt(event.target.innerText, 10) || 0;
},


  validateAndFormatInput(event) {
    // Allow control keys such as arrows, delete, backspace
    const controlKeys = ['ArrowLeft', 'ArrowRight', 'Backspace', 'Delete', 'Enter'];
    if (controlKeys.includes(event.key)) return;

    // Check if it's a valid number, prevent input if not
    const regex = /^[0-9]*$/;
    if (!regex.test(event.key)) {
      event.preventDefault();
    }
  },
    getMark(subjectId, studentId) {
  let foundSubjectIndex = this.subjects.findIndex(subject => subject.id === subjectId);
  let foundStudentIndex = this.students.findIndex(student => student.id === studentId);
  // Return the mark based on found indices
  return this.marks[foundSubjectIndex][foundStudentIndex];
},

    sortRows(subjectId) {
  const subjectIndex = this.subjects.findIndex(subject => subject.id === subjectId);
  let isAscending = true;
  if (this.currentSort.type === 'subject' && this.currentSort.id === subjectId) {
    isAscending = this.currentSort.direction === 'descending';
  }
  this.currentSort = {
    type: 'subject',
    id: subjectId,
    direction: isAscending ? 'ascending' : 'descending',
  };

  // Create an array that pairs each mark with its corresponding student for the selected subject
  let pairedMarksAndStudents = this.marks[subjectIndex].map((mark, index) => ({
    mark,
    student: this.students[index],
  }));

  // Sort this array based on marks, in the specified direction
  pairedMarksAndStudents.sort((a, b) => isAscending ? a.mark - b.mark : b.mark - a.mark);

  // Apply the sorted order of students to a new students array
  const sortedStudents = pairedMarksAndStudents.map(item => item.student);

  // Reconstruct the marks array to reflect the new order of students
  const newMarks = this.subjects.map(() => new Array(this.students.length));

  for (let i = 0; i < this.subjects.length; i++) {
    for (let j = 0; j < sortedStudents.length; j++) {
      const originalIndex = this.students.findIndex(s => s.id === sortedStudents[j].id);
      newMarks[i][j] = this.marks[i][originalIndex];
    }
  }

  // Update the students and marks arrays with the new order
  this.students = sortedStudents;
  this.marks = newMarks;
},

    sortColumns(studentId) {
  // Finds the index of the student to sort by
  const studentIndex = this.students.findIndex(student => student.id === studentId);
  
  // Determine sort direction
  let isAscending = true;
  if (this.currentSort.type === 'student' && this.currentSort.id === studentId) {
    // If the same studentId is clicked again, reverse the direction
    isAscending = this.currentSort.direction !== 'ascending';
  }
  this.currentSort = {
    type: 'student',
    id: studentId,
    direction: isAscending ? 'ascending' : 'descending',
  };

  const marksWithIndex = this.marks.map((marksArray, index) => {
    return { mark: marksArray[studentIndex], index };
  });

  // Sort this array based on marks, in the specified direction
  marksWithIndex.sort((a, b) => isAscending ? a.mark - b.mark : b.mark - a.mark);

  // Apply the sorted order of subjects to a new marks array
  const newMarks = marksWithIndex.map(sortedItem => this.marks[sortedItem.index]);

  // Update the marks array with the new order
  this.marks = newMarks;

  this.subjects = marksWithIndex.map(sortedItem => this.subjects[sortedItem.index]);
},

handleKeyDown(event, rowIndex, columnIndex, type, id) {
  const key = event.key;
  const rows = this.$el.querySelectorAll('table tr');
  
  let nextRow = rowIndex;
  let nextColumn = columnIndex;

  if (key === 'Enter' && !this.isEditMode) {
    // Call the toggleAndSort method when Enter is pressed
    console.log(`Type: ${type}, ID: ${id}`); 
    this.toggleAndSort(type, id);
    event.preventDefault(); 
    return; 
  }

   // Handle Enter key press to toggle edit mode for a cell
   if (key === 'Enter' && this.isEditMode) {
    // If the table is in edit mode, make the cell editable
    if(type !== 'subject' && type !== 'student') { 
      const cellId = `${id}-${this.students[columnIndex-1].id}`;
      this.editableCellId = this.editableCellId === cellId ? null : cellId;
      this.$nextTick(() => {
        const editableElement = this.$el.querySelector(`[data-cell-id="${this.editableCellId}"]`);
        if(editableElement) editableElement.focus(); // Focus on the newly editable cell
      });
    }
    event.preventDefault(); 
    return;
  }

   if (key === 'ArrowUp') {
    nextRow = Math.max(0, rowIndex - 1);
  } else if (key === 'ArrowDown') {
    nextRow = Math.min(rows.length - 1, rowIndex + 1);
  } else if (key === 'ArrowLeft') {
    nextColumn = Math.max(0, columnIndex - 1);
  } else if (key === 'ArrowRight') {
    nextColumn = Math.min(rows[rowIndex].children.length - 1, columnIndex + 1);
  } else {
    return; // Exit the function if it's not an arrow key
  }

  // Focus the next cell
  const nextCell = rows[nextRow].children[nextColumn];
  if (nextCell) {
    nextCell.focus();
  }

  event.preventDefault(); // Prevent the default arrow key behavior (scrolling)
},


    toggleAndSort(type, id) {
      console.log(type);

      if (type === 'subject') {
        this.sortRows(id);
      } else if (type === 'student') {
        this.sortColumns(id);
      }
    },

    onCellClick(subjectId, studentId) {
  if (!this.isEditMode) return; // Only proceed if in edit mode
  const cellId = `${subjectId}-${studentId}`;
  this.editableCellId = this.editableCellId === cellId ? null : cellId;
},

  },
};
</script>

<template>
  <div>
    <div class="visually-hidden" id="tableDescription">
  This table shows the marks obtained by students in different subjects. You can click on student names to sort by their performance, or on subjects to sort by marks in each subject.
    </div>

    <div class="table-responsive">
      <table aria-describedby="tableDescription" role="grid">
      <caption>Students' Marks by Subject</caption>

      <thead>
        <tr>
          <!-- Empty top-left corner cell -->
          <th></th>
          <!-- Horizontal Headers (Students) -->
          <th
            v-for="(student, index) in students"
            :key="`student-${student.id}`"
            :class="{'is-sorted': currentSort.id === student.id}"
            scope="col" 
            tabindex="0"
            role="button"
            aria-pressed="false"
            @click="sortColumns(student.id)"
            @keydown="handleKeyDown($event, 0, index + 1,'student', student.id)"          >
            {{ student.name }}
            <span v-if="currentSort.id === student.id">
              {{ currentSort.direction === 'ascending' ? '▲' : '▼' }}
            </span>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(subject, subjectIndex) in subjects" :key="`subject-${subject.id}`">
          <!-- Vertical Headers (Subjects) -->
          <td
            scope="row" 
            tabindex="0"
            role="button"
            aria-pressed="false"
            :class="{'is-sorted': currentSort.id === subject.id}"
            @click="sortRows(subject.id)"
            @keydown="handleKeyDown($event, subjectIndex + 1, 0,'subject', subject.id)" 
          >
            {{ subject.name }}
            <span v-if="currentSort.id === subject.id">
              {{ currentSort.direction === 'ascending' ? '▲' : '▼' }}
            </span>
          </td>

          <!-- Marks -->
          <td v-for="(student, colIndex) in students" :key="`mark-${subject.id}-${student.id}`" role="gridcell" tabindex="0" :data-cell-id="`${subject.id}-${student.id}`" @keydown="handleKeyDown($event, subjectIndex + 1, colIndex + 1, '', `${subject.id}-${student.id}`)" :contenteditable="editableCellId === `${subject.id}-${student.id}`" @click="onCellClick(subject.id, student.id)" @blur="updateMark(subject.id, student.id, $event)" @keypress="validateAndFormatInput">
  {{ getMark(subject.id, student.id) }}
</td>





        </tr>
      </tbody>
    </table>

    <button @click="toggleEditMode">{{ isEditMode ? 'Disable Edit Mode' : 'Enable Edit Mode' }}</button>

    </div>
    
  </div>
  
</template>







<style scoped>


  table {
    width: 100%;
  }

  .table-responsive {
    overflow-x: auto;
  }

  .is-sorted {
    font-weight: bold;
    color: #007bff; 
  }


  .visually-hidden {
    position: absolute;
    width: 1px;
    height: 1px;
    margin: -1px;
    padding: 0;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    border: 0;
  }

  table {
    width: 100%;
    border-collapse: collapse;
  }

  th, td {
    text-align: left;
    padding: 8px;
    border: 1px solid #ddd;
  }

  th {
    background-color: #f2f2f2;
  }

  th:focus, td:focus {
    outline: 2px solid #005fcc; /* High contrast outline for focused elements */
    outline-offset: -1px;
  }

  tr:nth-child(even) {
    background-color: #f9f9f9;
  }

  input[type="text"] {
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 4px;
    width: 100%;
  }

  input[type="text"]:focus {
    border-color: #005fcc;
  }

  th:focus, td:focus {
    outline: 2px solid blue; }


</style>

