<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nova Denúncia - Secretaria de Meio Ambiente</title>
    
    <!-- CSS do Leaflet para o Mapa -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    
    <style>
        :root {
            --primary-color: #2e7d32;
            --secondary-color: #4caf50;
            --bg-color: #f1f8e9;
            --text-color: #333;
            --border-color: #ccc;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }

        .container {
            background-color: #fff;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 800px;
        }

        h2 {
            color: var(--primary-color);
            text-align: center;
            margin-top: 0;
            border-bottom: 2px solid var(--secondary-color);
            padding-bottom: 10px;
        }

        .form-group {
            margin-bottom: 20px;
        }

        label {
            display: block;
            font-weight: bold;
            margin-bottom: 8px;
            color: var(--primary-color);
        }

        input[type="text"], 
        textarea, 
        select {
            width: 100%;
            padding: 10px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            box-sizing: border-box;
            font-family: inherit;
        }

        textarea {
            resize: vertical;
            min-height: 80px;
        }

        /* Estilos do Mapa */
        #map {
            height: 350px;
            width: 100%;
            border-radius: 4px;
            border: 1px solid var(--border-color);
            margin-bottom: 10px;
        }

        .map-instruction {
            font-size: 0.85em;
            color: #666;
            margin-top: -5px;
            margin-bottom: 10px;
        }

        /* Upload de Imagens */
        input[type="file"] {
            display: none;
        }

        .file-upload-btn {
            display: inline-block;
            background-color: #e0e0e0;
            color: #333;
            padding: 10px 15px;
            border-radius: 4px;
            cursor: pointer;
            transition: background 0.3s;
            border: 1px solid #ccc;
        }

        .file-upload-btn:hover {
            background-color: #d5d5d5;
        }

        .image-preview {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
            margin-top: 15px;
        }

        .image-preview img {
            height: 100px;
            width: 100px;
            object-fit: cover;
            border-radius: 4px;
            border: 1px solid var(--border-color);
        }

        /* Botão de Envio */
        .submit-btn {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 15px;
            font-size: 16px;
            font-weight: bold;
            border-radius: 4px;
            cursor: pointer;
            width: 100%;
            transition: background 0.3s;
            margin-top: 10px;
        }

        .submit-btn:hover {
            background-color: #1b5e20;
        }
    </style>
</head>
<body>

    <div class="container">
        <h2>Registro de Denúncia Ambiental</h2>
        
        <form id="denunciaForm" action="#" method="POST" enctype="multipart/form-data">
            
            <!-- Tipo de Denúncia -->
            <div class="form-group">
                <label for="tipoDenuncia">Tipo de Infração</label>
                <select id="tipoDenuncia" name="tipoDenuncia" required>
                    <option value="" disabled selected>Selecione o tipo de denúncia...</option>
                    <option value="queimada">🔥 Queimada Irregular</option>
                    <option value="poluicao_sonora">🔊 Poluição Sonora</option>
                </select>
            </div>

            <!-- Mapa para Localização -->
            <div class="form-group">
                <label>Local da Ocorrência</label>
                <div id="map"></div>
                <div class="map-instruction">Clique no mapa para marcar a localização exata da denúncia.</div>
                
                <!-- Inputs ocultos para enviar as coordenadas ao backend -->
                <input type="hidden" id="latitude" name="latitude" required>
                <input type="hidden" id="longitude" name="longitude" required>
            </div>

            <!-- Endereço Descritivo -->
            <div class="form-group">
                <label for="enderecoDescricao">Pontos de Referência / Endereço</label>
                <input type="text" id="enderecoDescricao" name="enderecoDescricao" placeholder="Ex: Rua principal, próximo ao mercado da esquina..." required>
            </div>

            <!-- Descrição da Denúncia -->
            <div class="form-group">
                <label for="descricaoDenuncia">Descrição da Denúncia</label>
                <textarea id="descricaoDenuncia" name="descricaoDenuncia" placeholder="Detalhe a situação (ex: dias e horários que o problema ocorre, características do infrator, etc.)" required></textarea>
            </div>

            <!-- Anexo de Evidências (Fotos) -->
            <div class="form-group">
                <label>Evidências (Fotos)</label>
                <label for="fotos" class="file-upload-btn">📷 Adicionar Fotos</label>
                <input type="file" id="fotos" name="fotos" accept="image/*" multiple>
                <div class="image-preview" id="imagePreview"></div>
            </div>

            <button type="submit" class="submit-btn">Enviar Denúncia</button>
        </form>
    </div>

    <!-- JS do Leaflet para o Mapa -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    
    <script>
        // INICIALIZAÇÃO DO MAPA
        // Coordenadas iniciais configuradas para a região.
        const initialLat = -1.982;
        const initialLng = -47.943;
        const zoomLevel = 13;

        const map = L.map('map').setView([initialLat, initialLng], zoomLevel);

        // Camada do OpenStreetMap
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map);

        let marker;

        // Evento de clique para adicionar o marcador
        map.on('click', function(e) {
            const lat = e.latlng.lat;
            const lng = e.latlng.lng;

            // Remove o marcador anterior, se houver
            if (marker) {
                map.removeLayer(marker);
            }

            // Adiciona o novo marcador
            marker = L.marker([lat, lng]).addTo(map);

            // Preenche os campos ocultos do formulário
            document.getElementById('latitude').value = lat;
            document.getElementById('longitude').value = lng;
        });

        // PREVIEW DE IMAGENS
        const fotoInput = document.getElementById('fotos');
        const imagePreviewContainer = document.getElementById('imagePreview');

        fotoInput.addEventListener('change', function(e) {
            // Limpa o preview anterior
            imagePreviewContainer.innerHTML = '';
            
            const files = e.target.files;

            if (files) {
                Array.from(files).forEach(file => {
                    // Confirma se é uma imagem
                    if (file.type.match('image.*')) {
                        const reader = new FileReader();

                        reader.onload = function(event) {
                            const img = document.createElement('img');
                            img.src = event.target.result;
                            imagePreviewContainer.appendChild(img);
                        }

                        reader.readAsDataURL(file);
                    }
                });
            }
        });

        // INTERCEPTADOR DE SUBMIT (Para testes)
        document.getElementById('denunciaForm').addEventListener('submit', function(e) {
            e.preventDefault(); // Evita o envio real para fins de demonstração
            
            const lat = document.getElementById('latitude').value;
            const lng = document.getElementById('longitude').value;

            if(!lat || !lng) {
                alert("Por favor, marque o local da denúncia no mapa.");
                return;
            }

            alert("Denúncia registrada com sucesso!\nSimulação de envio das coordenadas: " + lat + ", " + lng);
            // Aqui você conectaria com o seu backend (ex: PHP, Node.js, banco de dados SQL via AJAX/Fetch).
        });
    </script>
</body>
</html>