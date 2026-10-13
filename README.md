# Health-worker-companion
Offline-first health worker companion prototype
!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Health Worker Companion</title>
<style>
* { box-sizing: border-box; }
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #f1f7f5;
  color: #18352f;
}
header {
  background: #087f68;
  color: white;
  padding: 22px 18px;
  border-radius: 0 0 22px 22px;
}
header h1 { font-size: 24px; margin: 0 0 8px; }
header p { margin: 0; font-size: 14px; }
main { max-width: 600px; margin: auto; padding: 16px; }
.card {
  background: white;
  padding: 18px;
  margin-bottom: 16px;
  border-radius: 16px;
  box-shadow: 0 3px 12px #18352f0c;
}
h2 { font-size: 18px; margin: 0 0 14px; }
label {
  display: block;
  font-size: 14px;
  font-weight: bold;
  margin: 12px 0 6px;
}
input, select, textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #cadbd5;
  border-radius: 9px;
  font-size: 16px;
  background: white;
}
textarea { min-height: 70px; resize: vertical; }
button {
  border: 0;
  padding: 12px 15px;
  border-radius: 9px;
  background: #087f68;
  color: white;
  font-size: 15px;
  font-weight: bold;
  cursor: pointer;
}
.primary { width: 100%; margin-top: 16px; }
.secondary {
  background: #e0f3ed;
  color: #075b4b;
}
.small { font-size: 12px; color: #5b716a; }
.stats { display: flex; gap: 10px; }
.stat {
  flex: 1;
  background: #e0f3ed;
  border-radius: 12px;
  padding: 13px;
}
.stat strong { display: block; font-size: 24px; }
.record {
  border: 1px solid #dce9e4;
  padding: 13px;
  border-radius: 12px;
  margin-top: 10px;
  overflow-wrap: anywhere;
}
.record h3 { margin: 0 0 8px; font-size: 16px; }
.record p { font-size: 14px; margin: 6px 0; }
.status { font-weight: bold; color: #087f68; }
.delete { background: #a83232; margin-top: 10px; }
footer { text-align: center; padding: 12px; }
</style>
</head>
<body>

<header>
  <h1>🌿 Health Worker Companion</h1>
  <p>Your village visits, organized in one place.</p>
  <p id="connection">Checking connection...</p>
</header>

<main>
  <section class="card">
    <h2>📊 Visit Dashboard</h2>
    <div class="stats">
      <div class="stat">
        <strong id="total">0</strong>
        <span>Total visits</span>
      </div>
      <div class="stat">
        <strong id="pending">0</strong>
        <span>Follow-ups due</span>
      </div>
    </div>
  </section>

  <section class="card">
    <h2>➕ Record a Household Visit</h2>
    <form id="visitForm">
      <label for="household">Household / Demo ID</label>
      <input id="household" required
        placeholder="Example: Demo Family 01">

      <label for="visitDate">Visit date</label>
      <input id="visitDate" type="date" required>

      <label for="followDate">Follow-up date</label>
      <input id="followDate" type="date">

      <label for="outcome">Visit status</label>
      <select id="outcome">
        <option>Visit completed</option>
        <option>Follow-up required</option>
        <option>Visit needs rescheduling</option>
      </select>

      <label for="notes">General notes (use demo data only)</label>
      <textarea id="notes"
        placeholder="Example: Arrange another household visit"></textarea>

      <button class="primary" type="submit">Save Visit</button>
    </form>
    <p class="small">
      Prototype only. Do not enter real patient information.
    </p>
  </section>

  <section class="card">
    <h2>📅 Household Follow-ups</h2>
    <input id="search" placeholder="Search demo household records...">
    <div id="records"></div>
  </section>

  <section class="card">
    <h2>💚 General Health Information</h2>
    <p><strong>English:</strong> Follow the guidance provided by
      qualified healthcare professionals. Seek urgent medical care
      when needed.</p>
    <p><strong>తెలుగు:</strong> అర్హత కలిగిన వైద్య నిపుణులు ఇచ్చిన
      సూచనలను పాటించండి. అవసరమైనప్పుడు వెంటనే వైద్య సహాయం పొందండి.</p>
    <p class="small">
      This information is general and does not replace medical advice.
    </p>
  </section>
</main>

<footer class="small">
  Health Worker Companion · Student Hackathon Prototype
</footer>

<>
const STORAGE_KEY = "healthWorkerDemoVisits";
const form = document.getElementById("visitForm");
const recordsBox = document.getElementById("records");
const search = document.getElementById("search");

function loadVisits() {
try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
  } catch (error) {
    return [];
  }
}

function saveVisits(visits) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(visits));
}

function render() {
  const visits = loadVisits();
  const today = new Date().toISOString().slice(0, 10);

  document.getElementById("total").textContent = visits.length;

const due = visits.filter(v =>
  v.followDate === today
).length;

document.getElementById("pending").textContent = due;
  const term = search.value.toLowerCase();
  const filtered = visits.filter(v =>
    v.household.toLowerCase().includes(term)
  );

  recordsBox.innerHTML = "";

  if (filtered.length === 0) {
    recordsBox.textContent = "No visits yet. Add a demo visit above.";
    return;
  }

  filtered.forEach(v => {
    const item = document.createElement("div");
    item.className = "record";

    const heading = document.createElement("h3");
    heading.textContent = "🏠 " + v.household;

    const visit = document.createElement("p");
    visit.textContent = "Visit date: " + v.visitDate;

    const follow = document.createElement("p");
    follow.textContent = "Follow-up: " + (v.followDate || "Not set");

    const todayDate = new Date();
    todayDate.setHours(0, 0, 0, 0);
    
    const followDate = v.followDate
    ? new Date(v.followDate + "T00:00:00")
    : null;

    const status = document.createElement("p");

    if (!followDate) {
    status.textContent = "Follow-up status: Not scheduled";
    } else if (followDate < todayDate) {
    status.textContent = "Follow-up status: Overdue";
    } else if (followDate.getTime() === todayDate.getTime()) {
  status.textContent = "Follow-up status: Due today";
} else {
    status.textContent = "Follow-up status: Upcoming";
    }

    const outcome = document.createElement("p");
    outcome.className = "status";
    outcome.textContent = v.outcome;

    const notes = document.createElement("p");
    notes.textContent = "Notes: " + (v.notes || "None");

    const del = document.createElement("button");
    del.className = "delete";
    del.textContent = "Delete demo record";
    del.addEventListener("click", () => {
      if (confirm("Delete this demo record?")) {
        saveVisits(loadVisits().filter(x => x.id !== v.id));
        render();
      }
    });

    item.append(heading, visit, follow, status, outcome, notes, del);
    recordsBox.appendChild(item);
  });
}

form.addEventListener("submit", event => {
  event.preventDefault();

  const visitDate = document.getElementById("visitDate").value;
  const followDate = document.getElementById("followDate").value;

  if (followDate && followDate < visitDate) {
    alert("Follow-up date cannot be before the visit date.");
    return;
  }

  const visits = loadVisits();

  visits.unshift({
    id: Date.now().toString(),
    household: document.getElementById("household").value.trim(),
    visitDate,
    followDate,
    outcome: document.getElementById("outcome").value,
    notes: document.getElementById("notes").value.trim()
  });

  try {
    saveVisits(visits);
    form.reset();
    document.getElementById("visitDate").value =
      new Date().toISOString().slice(0, 10);
    render();
    alert("Demo visit saved on this device!");
  } catch (error) {
    alert("Could not save the visit. Check browser storage.");
  }
});

search.addEventListener("input", render);

function updateConnection() {
  document.getElementById("connection").textContent =
    navigator.onLine ? "● Internet connected" : "● Offline mode";
}

window.addEventListener("online", updateConnection);
window.addEventListener("offline", updateConnection);

document.getElementById("visitDate").value =
  new Date().toISOString().slice(0, 10);

updateConnection();
render();

if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("./sw.js")
    .then(() => console.log("Offline support registered"))
    .catch(error => console.log("Registration error:", error));
}
</script/>
</body>
</html>
