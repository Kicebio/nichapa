<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ระบบเก็บคะแนนแบบฝึกหัด</title>
  <style>
    body {
      font-family: sans-serif;
      padding: 1rem;
      background-color: #f4f4f4;
    }
    h1 {
      text-align: center;
    }
    .controls {
      display: flex;
      flex-wrap: wrap;
      gap: 1rem;
      justify-content: center;
      margin-bottom: 1rem;
    }
    select, input[type="number"], button {
      padding: 0.5rem;
      font-size: 1rem;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 1rem;
    }
    th, td {
      border: 1px solid #ccc;
      text-align: center;
      padding: 0.3rem;
    }
    .green {
      background-color: #a0e7a0;
    }
    .red {
      background-color: #f7a8a8;
    }
    .summary {
      font-weight: bold;
    }
  </style>
</head>
<body>
  <h1>ระบบเก็บคะแนนแบบฝึกหัด บทที่ 1 ม.5/8</h1>
  <div class="controls">
    <input type="number" id="studentNumber" placeholder="เลขที่นักเรียน (1-40)" min="1" max="40" />
    <select id="exercisePage">
      <!-- 14 หน้า -->
    </select>
    <button onclick="submitExercise()">ส่ง</button>
    <button onclick="resetData()">รีเซ็ตทั้งหมด</button>
    <button onclick="exportToExcel()">ส่งออก Excel</button>
  </div>
  <table id="scoreTable">
    <thead>
      <tr>
        <th>เลขที่</th>
        <!-- 14 หน้า -->
      </tr>
    </thead>
    <tbody>
      <!-- 40 แถว -->
    </tbody>
  </table>

  <script>
    const totalPages = 14;
    const maxStudents = 40;
    const table = document.getElementById("scoreTable");
    const storageKey = "exerciseTrackerSimple";

    // สร้างตัวเลือกใน dropdown
    const exerciseSelect = document.getElementById("exercisePage");
    for (let p = 1; p <= totalPages; p++) {
      const option = document.createElement("option");
      option.value = `p${p}`;
      option.textContent = `หน้า ${p}`;
      exerciseSelect.appendChild(option);
    }

    // โหลดข้อมูลจาก localStorage หรือสร้างใหม่
    let data = JSON.parse(localStorage.getItem(storageKey)) || {};

    function saveData() {
      localStorage.setItem(storageKey, JSON.stringify(data));
    }

    function renderTable() {
      const thead = table.querySelector("thead tr");
      const tbody = table.querySelector("tbody");
      thead.innerHTML = `<th>เลขที่</th>`;
      for (let p = 1; p <= totalPages; p++) {
        const th = document.createElement("th");
        th.textContent = `หน้า ${p}`;
        thead.appendChild(th);
      }
      const sumTh = document.createElement("th");
      sumTh.textContent = "% รวม";
      thead.appendChild(sumTh);

      tbody.innerHTML = "";
      for (let i = 1; i <= maxStudents; i++) {
        const tr = document.createElement("tr");
        const tdNumber = document.createElement("td");
        tdNumber.textContent = i;
        tr.appendChild(tdNumber);

        let count = 0;
        for (let p = 1; p <= totalPages; p++) {
          const td = document.createElement("td");
          const key = `p${p}`;
          const studentKey = `${i}`;
          if (data[studentKey] && data[studentKey][key]) {
            td.classList.add("green");
            td.textContent = "✔";
            count++;
          } else {
            td.classList.add("red");
            td.textContent = "✘";
          }
          tr.appendChild(td);
        }

        const percentTd = document.createElement("td");
        const percent = Math.round((count / totalPages) * 100);
        percentTd.textContent = `${percent}%`;
        percentTd.className = "summary";
        tr.appendChild(percentTd);

        tbody.appendChild(tr);
      }
    }

    function submitExercise() {
      const number = parseInt(document.getElementById("studentNumber").value);
      const pageKey = exerciseSelect.value;

      if (number < 1 || number > maxStudents) {
        alert("กรุณากรอกเลขที่นักเรียนระหว่าง 1-40");
        return;
      }

      if (!data[number]) {
        data[number] = {};
      }
      data[number][pageKey] = true;
      saveData();
      renderTable();
    }

    function resetData() {
      if (confirm("ต้องการรีเซ็ตข้อมูลทั้งหมดจริงหรือไม่?")) {
        localStorage.removeItem(storageKey);
        data = {};
        renderTable();
      }
    }

    function exportToExcel() {
      let csv = "เลขที่";
      for (let p = 1; p <= totalPages; p++) {
        csv += `,หน้า ${p}`;
      }
      csv += ",% รวม\n";

      for (let i = 1; i <= maxStudents; i++) {
        let row = `${i}`;
        let count = 0;
        for (let p = 1; p <= totalPages; p++) {
          const key = `p${p}`;
          if (data[i] && data[i][key]) {
            row += ",✔";
            count++;
          } else {
            row += ",✘";
          }
        }
        const percent = Math.round((count / totalPages) * 100);
        row += `,${percent}%`;
        csv += row + "\n";
      }

      const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
      const link = document.createElement("a");
      link.setAttribute("href", URL.createObjectURL(blob));
      link.setAttribute("download", "คะแนนแบบฝึกหัด.csv");
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }

    renderTable();
  </script>
</body>
</html>
