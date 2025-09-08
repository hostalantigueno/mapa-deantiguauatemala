<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Mapa Interactivo Antigua Guatemala</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/qrcode/build/qrcode.min.js"></script>
  <style>
    body { margin:0; font-family: Arial, sans-serif; display:flex; height:100vh; }
    #map { flex:3; }
    #panel { flex:1; padding:10px; overflow:auto; background:#f7f7f7; border-left:1px solid #ccc; }
    h2 { font-size:16px; margin-top:0; }
    ul { padding:0; list-style:none; }
    li { margin-bottom:8px; }
    button { margin:4px 2px; padding:4px 8px; font-size:12px; cursor:pointer; }
    #qrcode { margin-top:20px; text-align:center; }
    #importFile { display:none; }
  </style>
</head>
<body>
  <div id="map"></div>
  <div id="panel">
    <h2>Sitios Turísticos</h2>
    <ul id="listaSitios"></ul>
    <button onclick="clearAll()">🗑️ Borrar todos</button>
    <button onclick="exportJSON()">📤 Exportar JSON</button>
    <button onclick="document.getElementById('importFile').click()">📥 Importar JSON</button>
    <input type="file" id="importFile" accept=".json" onchange="importJSON(event)">
    <div id="qrcode"></div>
  </div>

  <script>
    const map = L.map("map").setView([14.5595, -90.7336], 15);
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "&copy; OpenStreetMap contributors"
    }).addTo(map);

    let sitios = [];

    const sitiosIniciales = [
      { name:"Hotel Boutique Casa Rosa", latlng:[14.5586, -90.7286] },
      { name:"Casa Donya Green Café", latlng:[14.5587, -90.7288] },
      { name:"Hobitenango Oficina", latlng:[14.5590, -90.7292] },
      { name:"Ruinas Santa Teresa de Jesús", latlng:[14.5591, -90.7294] },
      { name:"Templo La Merced", latlng:[14.5610, -90.7312] },
      { name:"Restaurante El Sabor de mi Tierra", latlng:[14.5612, -90.7313] },
      { name:"Restaurante México Lindo", latlng:[14.5613, -90.7311] },
      { name:"Lurel Bistro Restaurante", latlng:[14.5614, -90.7314] },
      { name:"Hotel y Traviling", latlng:[14.5620, -90.7321] },
      { name:"Restaurante Sabor Chapín", latlng:[14.5621, -90.7322] },
      { name:"Puerta Negra Restaurante", latlng:[14.5622, -90.7323] },
      { name:"Agencia de Viajes - Club Viajero", latlng:[14.5623, -90.7324] },
      { name:"Getaway Tours", latlng:[14.5624, -90.7325] },
      { name:"Reli Travel Tours", latlng:[14.5625, -90.7326] },
      { name:"Restaurante Maya Papaya", latlng:[14.5626, -90.7327] },
      { name:"Casa Amarilla Tours", latlng:[14.5630, -90.7331] },
      { name:"Ruinas de la Recolección", latlng:[14.5632, -90.7332] },
      { name:"Té Panda/Pastelería", latlng:[14.5618, -90.7350] },
      { name:"Aver Quién Soy", latlng:[14.5615, -90.7345] },
      { name:"La Cuevita de los Quesos", latlng:[14.5600, -90.7300] },
      { name:"Ruinas Las Capuchinas", latlng:[14.5602, -90.7301] },
      { name:"Ovce Cocina Vegana", latlng:[14.5588, -90.7287] },
      { name:"Los Colores de la Tierra (artículos típicos)", latlng:[14.5575, -90.7275] },
      { name:"7 Caldos Restaurante", latlng:[14.5576, -90.7276] },
      { name:"Restaurante Arrin Cuan", latlng:[14.5578, -90.7278] },
      { name:"Casa Colibrí Casa de Arte y Gastronomía", latlng:[14.5579, -90.7279] },
      { name:"Konbu Restaurante de Ramen", latlng:[14.5581, -90.7281] },
      { name:"Clio's Restaurante", latlng:[14.5582, -90.7282] },
      { name:"Mercado Artesanal El Carmen", latlng:[14.5584, -90.7284] },
      { name:"Mercado Artesanal Kaqchikel", latlng:[14.5585, -90.7285] },
      { name:"Cofefe Shop", latlng:[14.5587, -90.7287] },
      { name:"Tienda Artesanía de mi Tierra", latlng:[14.5588, -90.7288] },
      { name:"Del Puente Hamburguesas", latlng:[14.5590, -90.7290] },
      { name:"Drevery Restaurante", latlng:[14.5591, -90.7291] },
      { name:"La VID Coffee", latlng:[14.5595, -90.7295] },
      { name:"Ojalá Café-Bar", latlng:[14.5596, -90.7296] },
      { name:"Ecotours Checoj's", latlng:[14.5597, -90.7297] }
    ];

    function saveToLocal() {
      const data = sitios.map(s => ({ name: s.name, latlng: s.latlng }));
      localStorage.setItem("sitiosMapa", JSON.stringify(data));
    }

    function loadFromLocal() {
      const data = localStorage.getItem("sitiosMapa");
      if (data) {
        JSON.parse(data).forEach(s => addSitio(s.latlng, s.name));
      }
    }

    function addSitio(latlng, name=null) {
      if (!name) {
        name = prompt("Nombre del sitio:", "Nuevo sitio turístico");
        if (!name) return;
      }
      const marker = L.marker(latlng, { draggable:true }).addTo(map).bindPopup(name);

      marker.on("dblclick", () => {
        if (confirm("¿Eliminar este sitio?")) {
          removeSitio(sitios.findIndex(s => s.marker === marker));
        }
      });

      marker.on("dragend", () => {
        const sitio = sitios.find(s => s.marker === marker);
        if (sitio) {
          sitio.latlng = marker.getLatLng();
          saveToLocal();
        }
      });

      sitios.push({ name, latlng, marker });
      renderList();
      saveToLocal();
    }

    function editSitio(index) {
      const nuevoNombre = prompt("Editar nombre del sitio:", sitios[index].name);
      if (nuevoNombre) {
        sitios[index].name = nuevoNombre;
        sitios[index].marker.setPopupContent(nuevoNombre);
        renderList();
        saveToLocal();
      }
    }

    function removeSitio(index) {
      map.removeLayer(sitios[index].marker);
      sitios.splice(index,1);
      renderList();
      saveToLocal();
    }

    function clearAll() {
      if (confirm("¿Eliminar todos los sitios?")) {
        sitios.forEach(s => map.removeLayer(s.marker));
        sitios = [];
        renderList();
        saveToLocal();
      }
    }

    function renderList() {
      const ul = document.getElementById("listaSitios");
      ul.innerHTML = "";
      sitios.forEach((s,i) => {
        const li = document.createElement("li");
        li.textContent = s.name;
        const btnEdit = document.createElement("button");
        btnEdit.textContent = "✏️";
        btnEdit.onclick = () => editSitio(i);
        const btnDel = document.createElement("button");
        btnDel.textContent = "❌";
        btnDel.onclick = () => removeSitio(i);
        li.appendChild(btnEdit);
        li.appendChild(btnDel);
        ul.appendChild(li);
      });
    }

    // Exportar a JSON
    function exportJSON() {
      const data = sitios.map(s => ({ name: s.name, latlng: s.latlng }));
      const blob = new Blob([JSON.stringify(data,null,2)], { type:"application/json" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "sitios_antigua.json";
      a.click();
      URL.revokeObjectURL(url);
    }

    // Importar JSON
    function importJSON(event) {
      const file = event.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = e => {
        clearAll();
        const data = JSON.parse(e.target.result);
        data.forEach(s => addSitio(s.latlng, s.name));
      };
      reader.readAsText(file);
    }

    map.on("click", e => addSitio(e.latlng));

    loadFromLocal();
    if (sitios.length === 0) {
      sitiosIniciales.forEach(s => addSitio(s.latlng, s.name));
    }

    // QR escaneable
    const currentUrl = window.location.href;
    QRCode.toCanvas(document.createElement("canvas"), currentUrl, { width:150 }, (err, canvas) => {
      if (!err) {
        document.getElementById("qrcode").appendChild(canvas);
      }
    });
  </script>
</body>
</html>
