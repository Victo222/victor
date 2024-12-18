<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Horario Chileno</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @keyframes fadeOut {
            from { opacity: 1; }
            to { opacity: 0; }
        }
        .fade-out {
            animation: fadeOut 8s ease-out forwards;
        }
    </style>
</head>
<body class="min-h-screen bg-gradient-to-r from-blue-400 to-purple-500 p-8">
    <div class="container mx-auto bg-white rounded-lg shadow-lg p-6">
        <h1 class="text-4xl font-bold mb-6 text-center text-gray-800">Horarios </h1>
        <div id="current-time" class="text-3xl mb-8 text-center text-gray-700"></div>
        <table class="w-full">
            <thead>
                <tr class="bg-gray-100">
                    <th class="text-lg p-2 text-left">Número de máquina</th>
                    <th class="text-lg p-2 text-left">Hora de salida</th>
                </tr>
            </thead>
            <tbody id="machine-list"></tbody>
        </table>
    </div>

    <script>
        let machines = [
            { number: '009', departureTime: '16:15' },
            { number: '015', departureTime: '16:16' },
            { number: '023', departureTime: '16:18' },
            { number: '007', departureTime: '12:18' },
            { number: '031', departureTime: '12:10' },
            { number: '002', departureTime: '12:12' },
            { number: '004', departureTime: '12:14' },
            { number: '005', departureTime: '12:16' },
            { number: '011', departureTime: '12:18' },
            { number: '018', departureTime: '12:20' },
            { number: '025', departureTime: '12:22' },
            { number: '033', departureTime: '12:24' },
            { number: '040', departureTime: '12:06' },
            { number: '047', departureTime: '12:08' },
            { number: '054', departureTime: '12:30' },
            { number: '061', departureTime: '12:32' },
            { number: '068', departureTime: '12:34' },
            { number: '075', departureTime: '12:36' },
            { number: '082', departureTime: '12:38' }
        ];

        function formatTime(date) {
            return date.toLocaleTimeString('es-CL', { 
                timeZone: 'America/Santiago',
                hour: '2-digit', 
                minute: '2-digit', 
                second: '2-digit',
                hour12: false
            });
        }

        function isAlertTime(departureTime) {
            const [hours, minutes] = departureTime.split(':').map(Number);
            const now = new Date();
            const departureDate = new Date(now.getFullYear(), now.getMonth(), now.getDate(), hours, minutes);
            const timeDiff = departureDate.getTime() - now.getTime();
            return {
                showAlert: timeDiff <= 60000 && timeDiff > 0,
                isImmediate: timeDiff <= 60000 && timeDiff > 0
            };
        }

        function updateCurrentTime() {
            const currentTimeElement = document.getElementById('current-time');
            currentTimeElement.textContent = `Hora actual: ${formatTime(new Date())}`;
        }

        function renderMachines() {
            const machineListElement = document.getElementById('machine-list');
            machineListElement.innerHTML = '';

            machines.forEach(machine => {
                const row = document.createElement('tr');
                row.className = 'hover:bg-gray-50';
                row.innerHTML = `
                    <td class="text-lg p-2">${machine.number}</td>
                    <td class="text-lg p-2 flex items-center">
                        <span class="departure-time" data-time="${machine.departureTime}" data-machine="${machine.number}">${machine.departureTime}</span>
                    </td>
                `;
                machineListElement.appendChild(row);
            });
        }

        function updateAlerts() {
            const departureTimeElements = document.querySelectorAll('.departure-time');
            departureTimeElements.forEach(element => {
                const departureTime = element.dataset.time;
                const { showAlert, isImmediate } = isAlertTime(departureTime);
                
                element.classList.toggle('text-red-600', isImmediate);
                element.classList.toggle('font-bold', isImmediate);

                const alertIcon = element.nextElementSibling;
                if (showAlert && !alertIcon) {
                    const icon = document.createElement('span');
                    icon.className = 'ml-2 text-yellow-500';
                    icon.innerHTML = '&#9888;'; // Unicode for warning triangle
                    element.parentNode.appendChild(icon);

                    // Announce the departure
                    const machineNumber = element.dataset.machine;
                    announceVoice(`Máquina ${machineNumber} sale a las ${departureTime}`);
                } else if (!showAlert && alertIcon) {
                    alertIcon.remove();
                }
            });
        }

        function checkDepartures() {
            const now = new Date();
            machines.forEach((machine, index) => {
                const [hours, minutes] = machine.departureTime.split(':').map(Number);
                const departureTime = new Date(now.getFullYear(), now.getMonth(), now.getDate(), hours, minutes);
                
                if (now >= departureTime) {
                    const row = document.querySelector(`#machine-list tr:nth-child(${index + 1})`);
                    if (row && !row.classList.contains('fade-out')) {
                        row.classList.add('fade-out');
                        setTimeout(() => {
                            machines = machines.filter(m => m.number !== machine.number);
                            renderMachines();
                        }, 8000);
                    }
                }
            });
        }

        function announceVoice(text) {
            if ('speechSynthesis' in window) {
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'es-CL'; // Set language to Chilean Spanish
                speechSynthesis.speak(utterance);
            }
        }

        function initialize() {
            renderMachines();
            setInterval(() => {
                updateCurrentTime();
                updateAlerts();
                checkDepartures();
            }, 1000);
        }

        initialize();
    </script>
</body>
</html>
