<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>我的追星追劇日記</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    h1 { color: #444; }
    .diary-entry { 
      border: 1px solid #ccc; 
      border-radius: 8px; 
      padding: 10px; 
      margin: 10px 0; 
    }
    .delete-btn {
      background-color: #ff4d4d;
      color: white;
      border: none;
      border-radius: 5px;
      padding: 5px 10px;
      cursor: pointer;
      margin-top: 5px;
    }
    .delete-btn:hover { background-color: #cc0000; }
    .hidden { display: none; }
  </style>
</head>
<body>
  <h1>我的追星追劇日記</h1>

  <!-- 模式切換 -->
  <label>選擇模式：</label>
  <select id="modeSelect">
    <option value="concert">追星日記</option>
    <option value="drama">追劇日記</option>
  </select>

  <!-- 追星日記表單 -->
  <form id="concertForm">
    <h2>追星日記</h2>
    <label>日期：</label>
    <input type="date" id="concertDate" required><br><br>

    <label>心情日記：</label><br>
    <textarea id="concertContent" rows="4" cols="40" required></textarea><br><br>

    <label>插入圖片：</label>
    <input type="file" id="concertImage"><br><br>

    <label>演唱會票價：</label>
    <input type="number" id="concertPrice"><br><br>

    <label>演唱會地點：</label>
    <input type="text" id="concertLocation"><br><br>

    <label>演唱會座位：</label>
    <input type="text" id="concertSeat"><br><br>

    <button type="submit">新增追星日記</button>
  </form>

  <!-- 追劇日記表單 -->
  <form id="dramaForm" class="hidden">
    <h2>追劇日記</h2>
    <label>日期：</label>
    <input type="date" id="dramaDate" required><br><br>

    <label>心情日記：</label><br>
    <textarea id="dramaContent" rows="4" cols="40" required></textarea><br><br>

    <label>插入圖片：</label>
    <input type="file" id="dramaImage"><br><br>

    <label>劇名：</label>
    <input type="text" id="dramaName"><br><br>

    <label>集數：</label>
    <input type="text" id="dramaEpisode"><br><br>

    <button type="submit">新增追劇日記</button>
  </form>

  <h2>日記列表</h2>
  <div id="diaryList"></div>

  <script>
    const modeSelect = document.getElementById("modeSelect");
    const concertForm = document.getElementById("concertForm");
    const dramaForm = document.getElementById("dramaForm");
    const diaryList = document.getElementById("diaryList");

    // 載入已儲存的日記
    window.onload = () => {
      const savedDiaries = JSON.parse(localStorage.getItem("diaries")) || [];
      savedDiaries.forEach(entry => renderDiary(entry));
    };

    // 模式切換
    modeSelect.addEventListener("change", () => {
      if (modeSelect.value === "concert") {
        concertForm.classList.remove("hidden");
        dramaForm.classList.add("hidden");
      } else {
        dramaForm.classList.remove("hidden");
        concertForm.classList.add("hidden");
      }
    });

    // 追星日記提交
    concertForm.addEventListener("submit", function(e) {
      e.preventDefault();
      const entry = {
        type: "concert",
        date: document.getElementById("concertDate").value,
        content: document.getElementById("concertContent").value,
        price: document.getElementById("concertPrice").value,
        location: document.getElementById("concertLocation").value,
        seat: document.getElementById("concertSeat").value,
        image: document.getElementById("concertImage").files[0]
          ? URL.createObjectURL(document.getElementById("concertImage").files[0])
          : null
      };
      saveDiary(entry);
      renderDiary(entry);
      concertForm.reset();
    });

    // 追劇日記提交
    dramaForm.addEventListener("submit", function(e) {
      e.preventDefault();
      const entry = {
        type: "drama",
        date: document.getElementById("dramaDate").value,
        content: document.getElementById("dramaContent").value,
        dramaName: document.getElementById("dramaName").value,
        episode: document.getElementById("dramaEpisode").value,
        image: document.getElementById("dramaImage").files[0]
          ? URL.createObjectURL(document.getElementById("dramaImage").files[0])
          : null
      };
      saveDiary(entry);
      renderDiary(entry);
      dramaForm.reset();
    });

    // 儲存到 LocalStorage
    function saveDiary(entry) {
      const savedDiaries = JSON.parse(localStorage.getItem("diaries")) || [];
      savedDiaries.push(entry);
      localStorage.setItem("diaries", JSON.stringify(savedDiaries));
    }

    // 刪除日記
    function deleteDiary(entryDiv, entry) {
      diaryList.removeChild(entryDiv);
      let savedDiaries = JSON.parse(localStorage.getItem("diaries")) || [];
      savedDiaries = savedDiaries.filter(d => !(d.date === entry.date && d.content === entry.content));
      localStorage.setItem("diaries", JSON.stringify(savedDiaries));
    }

    // 顯示日記
    function renderDiary(entry) {
      const entryDiv = document.createElement("div");
      entryDiv.className = "diary-entry";

      let html = "";
      if (entry.type === "concert") {
        html += `<h3>🎤 追星日記</h3><strong>${entry.date}</strong><p>${entry.content}</p>`;
        if (entry.image) html += `<img src="${entry.image}" width="200"><br>`;
        if (entry.price) html += `<p>票價：${entry.price} 元</p>`;
        if (entry.location) html += `<p>地點：${entry.location}</p>`;
        if (entry.seat) html += `<p>座位：${entry.seat}</p>`;
      } else {
        html += `<h3>📺 追劇日記</h3><strong>${entry.date}</strong><p>${entry.content}</p>`;
        if (entry.image) html += `<img src="${entry.image}" width="200"><br>`;
        if (entry.dramaName) html += `<p>劇名：${entry.dramaName}</p>`;
        if (entry.episode) html += `<p>集數：${entry.episode}</p>`;
      }

      const deleteBtn = document.createElement("button");
      deleteBtn.innerText = "刪除日記";
      deleteBtn.className = "delete-btn";
      deleteBtn.addEventListener("click", () => deleteDiary(entryDiv, entry));

      entryDiv.innerHTML = html;
      entryDiv.appendChild(deleteBtn);
      diaryList.appendChild(entryDiv);
    }
  </script>
</body>
</html>
