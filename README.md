[index.html.txt](https://github.com/user-attachments/files/23348095/index.html.txt)
<!doctype html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>����� ����� �������� (������)</title>
  <style>
    :root{--accent:#0b74de;--bg:#f7f7f8}
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial; background:var(--bg); margin:0; color:#111}
    .container{max-width:760px;margin:0 auto;padding:16px}
    header{display:flex;gap:12px;align-items:center;justify-content:space-between}
    h1{font-size:18px;margin:0}
    .btn{background:var(--accent);color:#fff;border:none;padding:12px;border-radius:10px;flex:1;margin:6px 0;font-weight:600}
    .btn.secondary{background:#e9e9ea;color:#111}
    .card{background:#fff;border-radius:14px;padding:12px;margin-top:12px;box-shadow:0 6px 18px rgba(8,11,20,0.06)}
    label{font-size:13px;display:block;margin-bottom:6px}
    input[type=text], input[type=tel], textarea, input[type=number], select{width:100%;padding:10px;border-radius:8px;border:1px solid #e3e3e6;font-size:14px;box-sizing:border-box}
    textarea{min-height:70px}
    .row{display:flex;gap:8px}
    .col{flex:1}
    table{width:100%;border-collapse:collapse;margin-top:8px}
    th,td{padding:6px 8px;border-bottom:1px solid #f0f0f2;font-size:13px;text-align:right}
    th{font-weight:700}
    .small{font-size:12px;color:#666}
    .total{display:flex;justify-content:space-between;align-items:center;padding:12px 8px;font-weight:700;font-size:16px}
    .footer-actions{display:flex;gap:8px;margin-top:12px}
    .actions-top{display:flex;gap:8px}
    .order-item-input{width:100%}
    .status-pill{padding:6px 10px;border-radius:999px;display:inline-block;font-size:13px}
    @media(min-width:520px){h1{font-size:20px}}
  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>���� ����� �������� � ����� ����</h1>
      <div style="min-width:140px;display:flex;gap:6px">
        <button id="btnNew" class="btn">����� ��� ����</button>
        <button id="btnList" class="btn secondary">��� �������</button>
      </div>
    </header>

    <!-- ���� ����� ��� -->
    <div id="pageNew" class="card" style="display:none">
      <form id="orderForm">
        <div class="row">
          <div class="col">
            <label>��� �������� *</label>
            <input type="text" id="pharmacyName" required placeholder="����: ������ ������" />
          </div>
          <div class="col">
            <label>��� ������ *</label>
            <input type="tel" id="phone" required placeholder="09xxxxxxxx" />
          </div>
        </div>

        <div class="row" style="margin-top:8px">
          <div class="col">
            <label>������� *</label>
            <input type="text" id="address" required placeholder="������ɡ ������..." />
          </div>
          <div class="col">
            <label>��� ������� *</label>
            <input type="text" id="repName" required placeholder="��� �������" />
          </div>
        </div>

        <div style="margin-top:8px">
          <label>��������� *</label>
          <textarea id="notes" required placeholder="������� �� �����"></textarea>
        </div>

        <div style="margin-top:12px">
          <label>������ ������ (16 ����)</label>
          <div class="card" style="padding:8px">
            <table id="itemsTable">
              <thead>
                <tr>
                  <th style="width:40%">��� ������</th>
                  <th style="width:20%">��� ����� (����)</th>
                  <th style="width:20%">�����</th>
                  <th style="width:20%">�������</th>
                </tr>
              </thead>
              <tbody>
                <!-- 16 �� �������� -->
              </tbody>
            </table>
          </div>
        </div>

        <div class="total" id="totalRow">
          <div>������� �����</div>
          <div id="grandTotal">0.00</div>
        </div>

        <div class="footer-actions">
          <button type="button" id="btnSendWhats" class="btn">����� ����� ��� ������</button>
          <button type="submit" class="btn secondary">��� ����� ������</button>
        </div>
      </form>
    </div>

    <!-- ���� ��� ������� -->
    <div id="pageList" class="card" style="display:none">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <h2 style="margin:0;font-size:16px">���� �������</h2>
        <div class="small">���� ��� ���� ����� ��������</div>
      </div>
      <div id="ordersList" style="margin-top:8px"></div>
      <div style="margin-top:12px;display:flex;gap:8px">
        <button id="btnClearAll" class="btn secondary">��� �� �������</button>
      </div>
    </div>

    <p class="small" style="margin-top:12px">������: �������� ���� ������ �� ������� (localStorage). �� "����� ��� ������" ���� ������ ���/������ �� �� �����.</p>
  </div>

  <script>
    // �����
    const btnNew = document.getElementById('btnNew');
    const btnList = document.getElementById('btnList');
    const pageNew = document.getElementById('pageNew');
    const pageList = document.getElementById('pageList');
    const itemsTableBody = document.querySelector('#itemsTable tbody');
    const grandTotalEl = document.getElementById('grandTotal');
    const orderForm = document.getElementById('orderForm');
    const btnSendWhats = document.getElementById('btnSendWhats');
    const ordersList = document.getElementById('ordersList');
    const btnClearAll = document.getElementById('btnClearAll');

    // �����
    const STATUSES = ['��� ��������','����','��� �������','�� �������','����'];

    // ����� 16 ��
    function createRows(){
      itemsTableBody.innerHTML = '';
      for(let i=0;i<16;i++){
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td><input class="order-item-input" data-field="name" type="text" placeholder="��� ������ ${i+1}" /></td>
          <td><input class="order-item-input" data-field="price" type="number" min="0" step="0.01" placeholder="0.00" /></td>
          <td><input class="order-item-input" data-field="qty" type="number" min="0" step="1" placeholder="0" /></td>
          <td class="small" data-field="lineTotal">0.00</td>
        `;
        itemsTableBody.appendChild(tr);
      }
      attachInputsListener();
      recalcTotal();
    }

    function attachInputsListener(){
      const inputs = itemsTableBody.querySelectorAll('input');
      inputs.forEach(inp=>{
        inp.addEventListener('input',()=>{
          const row = inp.closest('tr');
          const price = parseFloat(row.querySelector('input[data-field="price"]').value||0);
          const qty = parseFloat(row.querySelector('input[data-field="qty"]').value||0);
          const lineTotal = (price*qty) || 0;
          row.querySelector('[data-field="lineTotal"]').textContent = lineTotal.toFixed(2);
          recalcTotal();
        });
      });
    }

    function recalcTotal(){
      let sum = 0;
      itemsTableBody.querySelectorAll('tr').forEach(r=>{
        const t = parseFloat(r.querySelector('[data-field="lineTotal"]').textContent||0) || 0;
        sum += t;
      });
      grandTotalEl.textContent = sum.toFixed(2);
    }

    // ���� ������
    btnNew.addEventListener('click',()=>{ pageNew.style.display='block'; pageList.style.display='none'; });
    btnList.addEventListener('click',()=>{ pageNew.style.display='none'; pageList.style.display='block'; renderOrders(); });

    // ��� ���
    orderForm.addEventListener('submit', (e)=>{
      e.preventDefault();
      const order = collectOrderFromForm();
      if(!order) return;
      saveOrder(order);
      alert('�� ��� ����� ������');
      orderForm.reset();
      createRows();
    });

    // ����� ��������
    function collectOrderFromForm(){
      const pharmacyName = document.getElementById('pharmacyName').value.trim();
      const phone = document.getElementById('phone').value.trim();
      const address = document.getElementById('address').value.trim();
      const repName = document.getElementById('repName').value.trim();
      const notes = document.getElementById('notes').value.trim();
      if(!pharmacyName||!phone||!address||!repName||!notes){ alert('������ ����� ���� ������ �������� (������ ������)'); return null; }

      const items = [];
      itemsTableBody.querySelectorAll('tr').forEach((r, idx)=>{
        const name = r.querySelector('input[data-field="name"]').value.trim();
        const price = parseFloat(r.querySelector('input[data-field="price"]').value||0);
        const qty = parseInt(r.querySelector('input[data-field="qty"]').value||0);
        if(name && qty>0){ items.push({name,price,qty,lineTotal: (price*qty)}); }
      });

      const total = parseFloat(grandTotalEl.textContent||0);
      if(items.length===0){ if(!confirm('�� ��� ������ �� ���� (�� ������ ����� �� ������� ���). �� ���� ��� ����� ���� ���Ͽ')) return null; }

      const order = {id: 'ord_' + Date.now(), pharmacyName, phone, address, repName, notes, items, total, status: STATUSES[0], createdAt: new Date().toISOString() };
      return order;
    }

    function saveOrder(order){
      const arr = JSON.parse(localStorage.getItem('orders')||'[]');
      arr.unshift(order);
      localStorage.setItem('orders', JSON.stringify(arr));
    }

    // ����� ��� ������
    btnSendWhats.addEventListener('click',()=>{
      const order = collectOrderFromForm();
      if(!order) return;
      const waText = buildWhatsAppText(order);
      const encoded = encodeURIComponent(waText);
      // ���� ����� ������ (web �� mobile)
      const waUrl = 'https://wa.me/?text=' + encoded;
      window.open(waUrl, '_blank');
    });

    function buildWhatsAppText(order){
      let lines = [];
      lines.push('��� ���� ��: ' + order.pharmacyName);
      lines.push('������: ' + order.phone);
      lines.push('�������: ' + order.address);
      lines.push('��� �������: ' + order.repName);
      lines.push('�������: ' + order.notes);
      lines.push('---');
      if(order.items && order.items.length){
        order.items.forEach((it, i)=>{
          lines.push(`${i+1}. ${it.name} - �����: ${it.price.toFixed(2)} - ������: ${it.qty} - �������: ${ (it.lineTotal).toFixed(2)}`);
        });
      } else {
        lines.push('�� ���� ���� �����');
      }
      lines.push('---');
      lines.push('������� �����: ' + order.total.toFixed(2));
      lines.push('����� �����: ' + new Date(order.createdAt).toLocaleString());
      return lines.join('\n');
    }

    // ��� �������
    function renderOrders(){
      const arr = JSON.parse(localStorage.getItem('orders')||'[]');
      if(arr.length===0){ ordersList.innerHTML = '<div class="small">�� ���� ����� ������.</div>'; return; }
      ordersList.innerHTML = '';
      arr.forEach(order=>{
        const div = document.createElement('div');
        div.className = 'card';
        const created = new Date(order.createdAt).toLocaleString();
        div.innerHTML = `
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div><strong>${escapeHtml(order.pharmacyName)}</strong><div class="small">${escapeHtml(order.phone)} � ${escapeHtml(created)}</div></div>
            <div style="text-align:left">
              <select data-id="${order.id}">
                ${STATUSES.map(s=>`<option ${s===order.status? 'selected': ''}>${s}</option>`).join('')}
              </select>
            </div>
          </div>
          <div class="small" style="margin-top:8px">${escapeHtml(order.address)}</div>
          <div style="margin-top:8px;font-size:14px">
            <div><strong>�������:</strong> ${order.total.toFixed(2)}</div>
            <div style="margin-top:6px"><button class="btn" data-action="view" data-id="${order.id}">��� ��������</button>
            <button class="btn secondary" data-action="wa" data-id="${order.id}">����� ��� ������</button></div>
          </div>
        `;
        ordersList.appendChild(div);
      });

      // ��� �������
      ordersList.querySelectorAll('select').forEach(sel=>{
        sel.addEventListener('change',(e)=>{
          const id = sel.getAttribute('data-id');
          updateOrderStatus(id, sel.value);
        });
      });
      ordersList.querySelectorAll('button[data-action]').forEach(b=>{
        b.addEventListener('click',(e)=>{
          const id = b.getAttribute('data-id');
          const action = b.getAttribute('data-action');
          if(action==='view') showOrderDetails(id);
          if(action==='wa') sendOrderWhatsById(id);
        });
      });
    }

    function escapeHtml(str){ if(!str) return ''; return String(str).replace(/[&<>"']/g, function(m){ return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":"&#39;"}[m]; }); }

    function updateOrderStatus(id, newStatus){
      const arr = JSON.parse(localStorage.getItem('orders')||'[]');
      const idx = arr.findIndex(o=>o.id===id);
      if(idx===-1) return;
      arr[idx].status = newStatus;
      localStorage.setItem('orders', JSON.stringify(arr));
      renderOrders();
    }

    function showOrderDetails(id){
      const arr = JSON.parse(localStorage.getItem('orders')||'[]');
      const order = arr.find(o=>o.id===id);
      if(!order) return alert('�� ��� ��� �����.');
      // ��� �������� ���� ����
      let txt = `������ ����� - ${order.pharmacyName}\n������: ${order.phone}\n�������: ${order.address}\n�������: ${order.total.toFixed(2)}\n\n������:\n`;
      if(order.items && order.items.length){ order.items.forEach((it,i)=>{ txt += `${i+1}. ${it.name} - ${it.qty} � ${it.price.toFixed(2)} = ${it.lineTotal.toFixed(2)}\n`; }); } else { txt += '�� ���� ����\n'; }
      alert(txt);
    }

    function sendOrderWhatsById(id){
      const arr = JSON.parse(localStorage.getItem('orders')||'[]');
      const order = arr.find(o=>o.id===id);
      if(!order) return alert('�� ��� ��� �����.');
      const url = 'https://wa.me/?text=' + encodeURIComponent(buildWhatsAppText(order));
      window.open(url, '_blank');
    }

    btnClearAll.addEventListener('click',()=>{
      if(confirm('�� ���� ��� ���� ������� �������ɿ')){ localStorage.removeItem('orders'); renderOrders(); }
    });

    // ����� �����
    createRows();
    // ��� ������ ����������: new
    pageNew.style.display='block';
  </script>
</body>
</html>
