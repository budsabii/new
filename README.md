<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>ระบบ IT Support Ticket</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen p-6">
  <div class="max-w-7xl mx-auto space-y-8">
    <h1 class="text-3xl font-bold text-center text-gray-800">📨 ระบบ IT Support Ticket</h1>

    <!-- Form: เปิด Ticket -->
    <div class="bg-white p-6 rounded shadow">
      <h2 class="text-xl font-semibold mb-4">✍️ เปิด Ticket</h2>
      <form id="ticketForm" class="grid md:grid-cols-2 gap-4">
        <input name="SubmitterName" placeholder="ชื่อผู้แจ้ง" required class="p-2 border rounded" />
        <input name="SubmitterEmail" placeholder="อีเมลผู้แจ้ง" type="email" required class="p-2 border rounded" />
        <input name="Subject" placeholder="หัวข้อปัญหา" required class="p-2 border rounded col-span-full" />
        <textarea name="Details" placeholder="รายละเอียดปัญหา" rows="4" required class="p-2 border rounded col-span-full"></textarea>
        <button type="submit" class="col-span-full bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded">
          ส่ง Ticket
        </button>
      </form>
      <div id="successMsg" class="text-green-600 mt-4 hidden">✅ ส่งเรียบร้อย! Ticket ID: <span id="ticketId"></span></div>
    </div>

    <!-- Dashboard: Ticket Table -->
    <div class="bg-white p-6 rounded shadow">
      <h2 class="text-xl font-semibold mb-4">🎛️ รายการ Ticket</h2>
      <table class="w-full text-sm">
        <thead class="bg-blue-600 text-white">
          <tr>
            <th class="p-2 text-left">Ticket ID</th>
            <th class="p-2 text-left">วันที่</th>
            <th class="p-2 text-left">ผู้แจ้ง</th>
            <th class="p-2 text-left">หัวข้อ</th>
            <th class="p-2 text-left">สถานะ</th>
            <th class="p-2 text-left">ผู้รับผิดชอบ</th>
            <th class="p-2 text-left">จัดการ</th>
          </tr>
        </thead>
        <tbody id="ticketTable" class="divide-y"></tbody>
      </table>
    </div>
  </div>

  <!-- Modal -->
  <div id="modal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
    <div class="bg-white p-6 rounded w-full max-w-md">
      <h3 class="text-lg font-bold mb-2">🛠 จัดการ Ticket</h3>
      <p>Ticket ID: <span id="modalTicketId" class="font-mono"></span></p>
      <label class="block mt-3">สถานะ:
        <select id="modalStatus" class="w-full mt-1 border rounded p-1">
          <option>รอดำเนินการ</option>
          <option>กำลังดำเนินการ</option>
          <option>ปิดแล้ว</option>
        </select>
      </label>
      <label class="block mt-3">ผู้รับผิดชอบ:
        <input id="modalAssignee" class="w-full mt-1 border rounded p-1" />
      </label>
      <label class="block mt-3">หมายเหตุ:
        <textarea id="modalNotes" rows="3" class="w-full mt-1 border rounded p-1"></textarea>
      </label>
      <div class="flex justify-end space-x-2 mt-4">
        <button onclick="closeModal()" class="px-3 py-1 bg-gray-400 text-white rounded">ยกเลิก</button>
        <button onclick="saveTicket()" class="px-3 py-1 bg-blue-600 text-white rounded">บันทึก</button>
      </div>
    </div>
  </div>

  <script>
    const API_URL = 'https://script.google.com/macros/s/AKfycbwzOcAOekpvQhfuZzYBfImsAGGPJWuamViVtPPIfgPIkU3mBgo2qtJHo8DQFvNct75BEA/exec'
    let selectedTicketId = ''

    // Submit Form
    const form = document.getElementById('ticketForm')
    form.addEventListener('submit', async (e) => {
      e.preventDefault()
      const formData = new FormData(form)
      const data = Object.fromEntries(formData.entries())
      const res = await fetch(API_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      })
      const result = await res.json()
      if (result.success) {
        document.getElementById('ticketId').textContent = result.ticketId
        document.getElementById('successMsg').classList.remove('hidden')
        form.reset()
        loadTickets()
      }
    })

    // Load Tickets
    async function loadTickets() {
      const res = await fetch(API_URL)
      const tickets = await res.json()
      const tbody = document.getElementById('ticketTable')
      tbody.innerHTML = ''
      tickets.reverse().forEach(t => {
        const tr = document.createElement('tr')
        tr.innerHTML = `
          <td class="p-2">${t.TicketID}</td>
          <td class="p-2">${new Date(t.Timestamp).toLocaleString()}</td>
          <td class="p-2">${t.SubmitterName}</td>
          <td class="p-2">${t.Subject}</td>
          <td class="p-2">${t.Status}</td>
          <td class="p-2">${t.AssigneeUsername || '-'}</td>
          <td class="p-2">
            <button onclick="openModal('${t.TicketID}', '${t.Status}', '${t.AssigneeUsername || ''}', \`${t.ResolutionNotes || ''}\`)"
              class="text-sm bg-yellow-500 text-white rounded px-2 py-1">จัดการ</button>
          </td>
        `
        tbody.appendChild(tr)
      })
    }

    // Modal control
    function openModal(id, status, assignee, notes) {
      selectedTicketId = id
      document.getElementById('modalTicketId').textContent = id
      document.getElementById('modalStatus').value = status
      document.getElementById('modalAssignee').value = assignee
      document.getElementById('modalNotes').value = notes
      document.getElementById('modal').classList.remove('hidden')
      document.getElementById('modal').classList.add('flex')
    }

    function closeModal() {
      document.getElementById('modal').classList.add('hidden')
    }

    async function saveTicket() {
      const body = {
        TicketID: selectedTicketId,
        Status: document.getElementById('modalStatus').value,
        AssigneeUsername: document.getElementById('modalAssignee').value,
        ResolutionNotes: document.getElementById('modalNotes').value
      }
      const res = await fetch(API_URL, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(body)
      })
      const result = await res.json()
      if (result.success) {
        closeModal()
        loadTickets()
      }
    }

    // Initial load
    loadTickets()
  </script>
</body>
</html>
