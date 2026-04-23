<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Visitor Management System</title>

<style>
:root {
    --primary: #2563eb;
}

body {
    font-family: 'Segoe UI', sans-serif;
    background: url('https://images.unsplash.com/photo-1521791136064-7986c2920216') no-repeat center/cover;
    display: flex;
    justify-content: center;
    padding: 20px;
}

.container {
    backdrop-filter: blur(15px);
    background: rgba(255,255,255,0.9);
    padding: 2rem;
    border-radius: 16px;
    width: 100%;
    max-width: 950px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

h2, h3 {
    text-align: center;
    color: #1e3a8a;
}

input, select {
    width: 100%;
    padding: 12px;
    margin: 8px 0;
    border-radius: 8px;
    border: 1px solid #ccc;
}

button {
    padding: 10px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
}

.checkin-btn {
    width: 100%;
    background: linear-gradient(135deg, #2563eb, #1e40af);
    color: white;
    font-weight: bold;
}

.checkout-btn {
    background: #16a34a;
    color: white;
}

.capture-btn {
    width: 100%;
    background: black;
    color: white;
    margin-top: 10px;
}

table {
    width: 100%;
    margin-top: 15px;
    border-collapse: collapse;
}

th, td {
    border: 1px solid #ddd;
    padding: 8px;
    font-size: 0.85rem;
    text-align: center;
}

th {
    background: var(--primary);
    color: white;
}

.timestamp {
    text-align: center;
    font-size: 0.8rem;
}

.photo-box {
    border: 2px dashed #aaa;
    padding: 15px;
    text-align: center;
    border-radius: 10px;
    cursor: pointer;
}

video {
    width: 100%;
    border-radius: 10px;
    margin-top: 10px;
}

img.preview {
    width: 100px;
    margin-top: 10px;
    border-radius: 6px;
}
</style>
</head>

<body>

<div class="container">

<h2>Visitor Check-In</h2>

<form id="visitorForm">

<input type="text" id="visitorName" placeholder="Name" required>
<input type="tel" id="visitorPhone" placeholder="Phone Number" required>
<input type="email" id="visitorEmail" placeholder="Email" required>

<select id="Dropdown" required>
<option value="">Select Gender</option>
<option value="Male">Male</option>
<option value="Female">Female</option>
<option value="Transgender">Transgender</option>
<option value="Bisexual">Bisexual</option>
<option value="Prefer Not To Say">Prefer Not To Say</option>
</select>

<select id="deptDropdown" required>
<option value="">Select Department</option>
<option value="IT">IT</option>
<option value="HR">HR</option>
<option value="Finance">Finance</option>
<option value="Bisinusses">Bisinusses</option>
<option value="Graphics">Graphics</option>
<option value="Administrator">Administrator</option>
<option value="Reciptionist">Reciptionist</option>
<option value="Electrical">Electrical</option>
</select>

<select id="staffDropdown" disabled required></select>

<div class="photo-box" id="photoBox">
📷 Click to Upload Photo
<input type="file" id="photoInput" accept="image/*" hidden>
<div id="previewContainer"></div>
</div>

<video id="camera" autoplay></video>
<button type="button" class="capture-btn" onclick="capturePhoto()">📸 Capture Photo</button>

<button type="submit" class="checkin-btn">Check In</button>

<div class="timestamp" id="time"></div>

</form>

<h3>Visitor Log</h3>

<!-- SINGLE DATE FILTER -->
<div style="margin-top:10px;">
    <input type="date" id="filterDate">
</div>

<!-- DOWNLOAD BUTTON -->
<button onclick="downloadExcel()" class="checkin-btn" style="margin-top:10px;">
📥 Download Selected Date (CSV)
</button>

<table>
<thead>
<tr>
<th>Photo</th>
<th>Name</th>
<th>Phone</th>
<th>Email</th>
<th>Dept</th>
<th>Person</th>
<th>Check-In</th>
<th>Check-Out</th>
<th>Action</th>
</tr>
</thead>
<tbody id="visitorTable"></tbody>
</table>

</div>

<script>

const visitorForm = document.getElementById("visitorForm");
const visitorName = document.getElementById("visitorName");
const visitorPhone = document.getElementById("visitorPhone");
const visitorEmail = document.getElementById("visitorEmail");
const deptDropdown = document.getElementById("deptDropdown");
const staffDropdown = document.getElementById("staffDropdown");
const visitorTable = document.getElementById("visitorTable");
const timeDisplay = document.getElementById("time");
const photoInput = document.getElementById("photoInput");
const previewContainer = document.getElementById("previewContainer");
const photoBox = document.getElementById("photoBox");
const camera = document.getElementById("camera");

const staffDirectory = {
    IT: ["Rahul", "Sneha"],
    HR: ["Nidin", "Nivediya"],
    Finance: ["George", "Lakshmi"],
    Bisinusses: ["Navya","Arun"],
    Graphics: ["Anoop","Albin"],
    Administrator: ["Priya","Aamina"],
    Reciptionist: ["Nanditha"],
    Electrical: ["Hashir","Alan","Vinayak","Ajin"]
};

let photoData = "";

photoBox.addEventListener("click", () => photoInput.click());

deptDropdown.addEventListener("change", () => {
    const dept = deptDropdown.value;
    staffDropdown.innerHTML = "";

    if (dept) {
        staffDropdown.disabled = false;

        staffDirectory[dept].forEach(name => {
            let opt = document.createElement("option");
            opt.value = name;
            opt.textContent = name;
            staffDropdown.appendChild(opt);
        });
    } else {
        staffDropdown.disabled = true;
    }
});

setInterval(() => {
    timeDisplay.innerText = "System Time: " + new Date().toLocaleString();
}, 1000);

photoInput.addEventListener("change", () => {
    const file = photoInput.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
        photoData = e.target.result;
        previewContainer.innerHTML = `<img src="${photoData}" class="preview">`;
    };
    reader.readAsDataURL(file);
});

navigator.mediaDevices.getUserMedia({ video: true })
.then(stream => camera.srcObject = stream)
.catch(err => console.log("Camera error:", err));

function capturePhoto() {
    const canvas = document.createElement("canvas");
    canvas.width = camera.videoWidth;
    canvas.height = camera.videoHeight;

    const ctx = canvas.getContext("2d");
    ctx.drawImage(camera, 0, 0);

    photoData = canvas.toDataURL("image/png");
    previewContainer.innerHTML = `<img src="${photoData}" class="preview">`;
}

visitorForm.addEventListener("submit", (e) => {
    e.preventDefault();

    if (!deptDropdown.value || !staffDropdown.value) {
        alert("Select department and staff");
        return;
    }

    const visitor = {
        id: Date.now(),
        name: visitorName.value,
        phone: visitorPhone.value,
        email: visitorEmail.value,
        dept: deptDropdown.value,
        staff: staffDropdown.value,
        photo: photoData,
        checkin: new Date().toISOString(),
        checkout: null
    };

    let data = JSON.parse(localStorage.getItem("visitors")) || [];
    data.push(visitor);
    localStorage.setItem("visitors", JSON.stringify(data));

    loadVisitors();

    alert("✅ Checked In");

    visitorForm.reset();
    previewContainer.innerHTML = "";
    photoData = "";
    staffDropdown.innerHTML = "";
    staffDropdown.disabled = true;
});

function checkoutVisitor(id) {
    let data = JSON.parse(localStorage.getItem("visitors")) || [];

    data.forEach(v => {
        if (v.id === id && !v.checkout) {
            v.checkout = new Date().toISOString();
        }
    });

    localStorage.setItem("visitors", JSON.stringify(data));
    loadVisitors();
}

function loadVisitors() {
    const data = JSON.parse(localStorage.getItem("visitors")) || [];
    visitorTable.innerHTML = "";

    data.forEach(v => {
        const row = document.createElement("tr");

        row.innerHTML = `
            <td>${v.photo ? `<img src="${v.photo}" width="50">` : "-"}</td>
            <td>${v.name}</td>
            <td>${v.phone}</td>
            <td>${v.email}</td>
            <td>${v.dept}</td>
            <td>${v.staff}</td>
            <td>${new Date(v.checkin).toLocaleString()}</td>
            <td>${v.checkout ? new Date(v.checkout).toLocaleString() : "—"}</td>
            <td>
                <button class="checkout-btn"
                    onclick="checkoutVisitor(${v.id})"
                    ${v.checkout ? "disabled" : ""}>
                    ${v.checkout ? "Done" : "Check Out"}
                </button>
            </td>
        `;

        visitorTable.appendChild(row);
    });
}

function downloadExcel() {
    const data = JSON.parse(localStorage.getItem("visitors")) || [];
    const selectedDate = document.getElementById("filterDate").value;

    if (!selectedDate) {
        alert("Please select a date");
        return;
    }

    const start = new Date(selectedDate);
    const end = new Date(selectedDate + "T23:59:59");

    const filtered = data.filter(v => {
        const checkin = new Date(v.checkin);
        return checkin >= start && checkin <= end;
    });

    if (filtered.length === 0) {
        alert("No records found for selected date");
        return;
    }

    let csv = "Name,Phone,Email,Department,Person,CheckIn,CheckOut\n";

    filtered.forEach(v => {
        csv += `"${v.name}","${v.phone}","${v.email}","${v.dept}","${v.staff}","${new Date(v.checkin).toLocaleString()}","${v.checkout ? new Date(v.checkout).toLocaleString() : ""}"\n`;
    });

    const blob = new Blob([csv], { type: "text/csv" });
    const url = URL.createObjectURL(blob);

    const a = document.createElement("a");
    a.href = url;
    a.download = "Visitor_Log_Selected_Date.csv";
    a.click();

    URL.revokeObjectURL(url);
}

loadVisitors();

</script>

</body>
</html>
