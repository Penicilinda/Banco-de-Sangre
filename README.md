# Banco-de-Sangre# Banco-de-Sangre
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Asistente de IA para Documentos</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- PDF.js library from Mozilla -->
    <script src="https://mozilla.github.io/pdf.js/build/pdf.js"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #3498db;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <div class="container mx-auto p-4 sm:p-6 md:p-8 max-w-3xl">
        
        <header class="text-center mb-8">
            <h1 class="text-3xl sm:text-4xl font-bold text-red-700">Asistente de IA para Bancos de Sangre</h1>
            <p class="mt-2 text-lg text-gray-600">Carga un documento (.txt o .pdf) y haz preguntas sobre su contenido.</p>
        </header>

        <main class="bg-white p-6 sm:p-8 rounded-2xl shadow-lg">

            <!-- Step 1: File Upload -->
            <div class="mb-6">
                <h2 class="text-xl font-semibold mb-3 text-gray-700 border-b pb-2">Paso 1: Carga tu documento</h2>
                <p class="text-sm text-gray-500 mb-4">Sube un archivo de texto (.txt) o PDF con la información que la IA debe usar.</p>
                <div class="flex items-center justify-center w-full">
                    <label for="file-upload" class="flex flex-col items-center justify-center w-full h-48 border-2 border-gray-300 border-dashed rounded-lg cursor-pointer bg-gray-50 hover:bg-gray-100 transition-colors">
                        <div id="upload-prompt" class="flex flex-col items-center justify-center pt-5 pb-6">
                            <svg class="w-10 h-10 mb-3 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"></path></svg>
                            <p id="file-name" class="mb-2 text-sm text-gray-500"><span class="font-semibold">Haz clic para subir</span> o arrastra y suelta</p>
                            <p class="text-xs text-gray-500">Archivos .TXT o .PDF</p>
                        </div>
                         <div id="processing-pdf" class="hidden flex-col items-center justify-center">
                            <div class="loader"></div>
                            <p class="mt-4 text-gray-600 font-semibold">Procesando PDF...</p>
                        </div>
                        <input id="file-upload" type="file" class="hidden" accept=".txt,.pdf" />
                    </label>
                </div>
            </div>

            <!-- Step 2: Question Input -->
            <div class="mb-6">
                <h2 class="text-xl font-semibold mb-3 text-gray-700 border-b pb-2">Paso 2: Haz tu pregunta</h2>
                <p class="text-sm text-gray-500 mb-4">Una vez cargado el archivo, escribe la pregunta que tengas sobre el documento.</p>
                <textarea id="question-input" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-red-500 focus:border-red-500 transition-shadow" rows="3" placeholder="Ej: ¿Cuál es el procedimiento para la donación de plaquetas?" disabled></textarea>
                <button id="ask-button" class="mt-4 w-full bg-red-600 text-white font-bold py-3 px-4 rounded-lg hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition-transform transform hover:scale-105 disabled:bg-gray-400 disabled:cursor-not-allowed disabled:transform-none" disabled>
                    Preguntar a la IA
                </button>
            </div>

            <!-- Step 3: AI Response -->
            <div>
                <h2 class="text-xl font-semibold mb-3 text-gray-700 border-b pb-2">Paso 3: Respuesta de la IA</h2>
                <div id="response-container" class="mt-4 p-4 min-h-[100px] bg-gray-50 rounded-lg border border-gray-200">
                    <p id="response-placeholder" class="text-gray-500">La respuesta aparecerá aquí...</p>
                    <div id="loader" class="hidden mx-auto loader"></div>
                    <p id="response-text" class="text-gray-800 whitespace-pre-wrap"></p>
                </div>
            </div>

        </main>

        <footer class="text-center mt-8 text-sm text-gray-500">
            <p>Creado para facilitar el acceso a información crítica en bancos de sangre.</p>
        </footer>
    </div>

    <script>
        // Setup for PDF.js worker
        pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://mozilla.github.io/pdf.js/build/pdf.worker.js';

        const fileUpload = document.getElementById('file-upload');
        const fileNameDisplay = document.getElementById('file-name');
        const questionInput = document.getElementById('question-input');
        const askButton = document.getElementById('ask-button');
        const responseContainer = document.getElementById('response-container');
        const responsePlaceholder = document.getElementById('response-placeholder');
        const loader = document.getElementById('loader');
        const responseText = document.getElementById('response-text');
        const uploadPrompt = document.getElementById('upload-prompt');
        const processingPdf = document.getElementById('processing-pdf');

        let documentContent = '';

        // Handle file upload
        fileUpload.addEventListener('change', async (event) => {
            const file = event.target.files[0];
            if (!file) return;

            // Reset UI
            documentContent = '';
            questionInput.disabled = true;
            askButton.disabled = true;
            fileNameDisplay.classList.remove('text-green-600', 'text-red-600', 'font-semibold');


            if (file.type === 'text/plain') {
                const reader = new FileReader();
                reader.onload = (e) => {
                    documentContent = e.target.result;
                    showSuccess(`Archivo cargado: ${file.name}`);
                };
                reader.readAsText(file);
            } else if (file.type === 'application/pdf') {
                uploadPrompt.classList.add('hidden');
                processingPdf.classList.remove('hidden');
                try {
                    const reader = new FileReader();
                    reader.onload = async (e) => {
                        const typedarray = new Uint8Array(e.target.result);
                        const pdf = await pdfjsLib.getDocument(typedarray).promise;
                        let fullText = '';
                        for (let i = 1; i <= pdf.numPages; i++) {
                            const page = await pdf.getPage(i);
                            const textContent = await page.getTextContent();
                            fullText += textContent.items.map(item => item.str).join(' ');
                        }
                        documentContent = fullText;
                        showSuccess(`PDF cargado y procesado: ${file.name}`);
                    };
                    reader.readAsArrayBuffer(file);
                } catch (error) {
                    console.error('Error processing PDF:', error);
                    showError('Error al procesar el archivo PDF.');
                } finally {
                    uploadPrompt.classList.remove('hidden');
                    processingPdf.classList.add('hidden');
                }
            } else {
                showError('Error: Formato de archivo no soportado.');
            }
        });

        function showSuccess(message) {
            fileNameDisplay.textContent = message;
            fileNameDisplay.classList.add('text-green-600', 'font-semibold');
            questionInput.disabled = false;
            askButton.disabled = false;
            questionInput.focus();
        }

        function showError(message) {
            fileNameDisplay.textContent = message;
            fileNameDisplay.classList.add('text-red-600', 'font-semibold');
            documentContent = '';
            questionInput.disabled = true;
            askButton.disabled = true;
        }


        // Handle ask button click
        askButton.addEventListener('click', async () => {
            const question = questionInput.value.trim();
            if (!question) {
                alert('Por favor, escribe una pregunta.');
                return;
            }
            if (!documentContent) {
                alert('Por favor, carga un documento primero.');
                return;
            }

            // Show loader and hide previous response
            loader.classList.remove('hidden');
            responsePlaceholder.classList.add('hidden');
            responseText.textContent = '';
            askButton.disabled = true;
            questionInput.disabled = true;
            
            try {
                // Construct the prompt for the AI
                const prompt = `Basado exclusivamente en el siguiente documento, responde la pregunta de forma concisa y clara. Si la respuesta no está en el texto, indica que no se encontró la información.\n\n--- INICIO DEL DOCUMENTO ---\n${documentContent}\n--- FIN DEL DOCUMENTO ---\n\nPregunta: ${question}\n\nRespuesta:`;
                
                // In a real application, this would be a fetch call to an AI service like Gemini.
                // For this example, we will continue to simulate a response based on keyword matching.
                const simulatedResponse = await getAIResponse(prompt);
                
                responseText.textContent = simulatedResponse;

            } catch (error) {
                console.error('Error getting AI response:', error);
                responseText.textContent = 'Hubo un error al procesar tu pregunta. Por favor, intenta de nuevo.';
            } finally {
                // Hide loader and enable inputs
                loader.classList.add('hidden');
                askButton.disabled = false;
                questionInput.disabled = false;
                questionInput.focus();
            }
        });
        
        // This function simulates a call to a generative AI.
        async function getAIResponse(prompt) {
             const questionWords = prompt.split('Pregunta:')[1].toLowerCase().match(/\b(\w+)\b/g) || [];
             const sentences = documentContent.split('.');
             let relevantSentences = [];
             
             sentences.forEach(sentence => {
                 for(const word of questionWords) {
                     // Find relevant sentences based on keywords
                     if(word.length > 3 && sentence.toLowerCase().includes(word)) {
                         relevantSentences.push(sentence.trim());
                         break; 
                     }
                 }
             });

            return new Promise(resolve => {
                setTimeout(() => {
                    if (relevantSentences.length > 0) {
                         // Join the most relevant sentences for the answer.
                         resolve("Basado en el documento, aquí está la información relevante:\n\n- " + relevantSentences.slice(0, 5).join('.\n- ') + '.');
                    } else {
                         resolve("No pude encontrar una respuesta directa a tu pregunta en el documento proporcionado. Por favor, intenta reformular tu pregunta o asegúrate de que el documento contenga la información.");
                    }
                }, 1500); // Simulate network delay
            });
        }

    </script>
</body>
</html>
