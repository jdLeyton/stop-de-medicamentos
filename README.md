<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>STOP Medicamentos (API + Offline)</title>
    <style>
        /* Estilos anteriores... */
        .connection-status {
            padding: 5px;
            border-radius: 3px;
            font-weight: bold;
        }
        .online { background-color: #2ecc71; color: white; }
        .offline { background-color: #e74c3c; color: white; }
    </style>
</head>
<body>
    <div class="container">
        <h1>💊 STOP Medicamentos <span id="connectionStatus" class="connection-status offline">Offline</span></h1>
        <!-- Contenido anterior... -->
    </div>

    <script>
        // Base de datos local ampliada (ejemplos colombianos)
        const database = {
            principios: ["Amoxicilina", "Ampicilina", "Azitromicina", "Albendazol", "Aciclovir", /*... +50 más */],
            grupos: ["Antibiótico", "Antiparasitario", "Antiviral", "Antihipertensivo", /*...*/],
            laboratorios: ["Genfar", "Tecnoquímicas", "Procaps", "Bayer", /*...*/],
            formas: ["Tableta", "Cápsula", "Suspensión", "Crema", /*...*/]
        };

        // URLs de APIs (ejemplos)
        const API_URLS = {
            principios: "https://api.medicamentos.com.co/principios?q=",
            grupos: "https://api.medicamentos.com.co/grupos?q="
        };

        let isOnline = false;

        // Verificar conexión
        function checkConnection() {
            isOnline = navigator.onLine;
            document.getElementById('connectionStatus').className = isOnline ? 
                'connection-status online' : 'connection-status offline';
            document.getElementById('connectionStatus').textContent = isOnline ? 'Online' : 'Offline';
        }

        // Función mejorada de validación
        async function validateAnswer(category, value) {
            if (!value || value[0].toUpperCase() !== currentLetter) return false;
            
            // 1. Primero verificar en base local (rápido)
            const localMatch = database[category].some(
                item => item.toLowerCase() === value.toLowerCase()
            );
            if (localMatch) return true;
            
            // 2. Si hay conexión, consultar API
            if (isOnline) {
                try {
                    const response = await fetch(`${API_URLS[category]}${encodeURIComponent(value)}`);
                    const data = await response.json();
                    return data.exists; // Suponiendo que la API devuelve {exists: boolean}
                } catch (error) {
                    console.error(`Error API ${category}:`, error);
                    return false;
                }
            }
            
            return false;
        }

        // Modificar la función stopGame para usar validación async
        async function stopGame() {
            clearInterval(timer);
            document.getElementById('startBtn').disabled = false;
            document.getElementById('stopBtn').disabled = true;
            
            const categories = [
                { id: 'principio', name: 'Principio activo', key: 'principios' },
                { id: 'grupo', name: 'Grupo terapéutico', key: 'grupos' },
                { id: 'laboratorio', name: 'Laboratorio', key: 'laboratorios' },
                { id: 'forma', name: 'Forma farmacéutica', key: 'formas' }
            ];
            
            let roundPoints = 0;
            let resultsHTML = '<h3>Resultados:</h3>';
            
            // Validar cada categoría
            for (const category of categories) {
                const input = document.getElementById(category.id);
                const value = input.value.trim();
                const isValid = await validateAnswer(category.key, value);
                
                resultsHTML += `
                    <p><strong>${category.name}:</strong> ${value} 
                    <span class="${isValid ? 'valid' : 'invalid'}">
                        ${isValid ? '✅ +10' : '❌ +0'}
                    </span></p>
                `;
                
                if (isValid) roundPoints += 10;
            }
            
            points += roundPoints;
            document.getElementById('points').textContent = points;
            document.getElementById('results').innerHTML = resultsHTML;
        }

        // Event listeners
        window.addEventListener('online', checkConnection);
        window.addEventListener('offline', checkConnection);
        checkConnection(); // Verificar al cargar
    </script>
</body>
</html>



// Ejemplo con API de OpenFDA (requiere registro)
const response = await fetch(`https://api.fda.gov/drug/label.json?search=openfda.substance_name:"${value}"&limit=1`);
const data = await response.json();
return data.results.length > 0;


// Ejemplo con API de INVIMA (simplificado)
const response = await fetch(`https://consultaregistro.invima.gov.co/medicamentos?nombre=${value}`);
return response.ok; // Verificar si la respuesta contiene datos
