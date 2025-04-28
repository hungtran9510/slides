<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Amazon Q Tester Assistant</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        .chat-container {
            height: calc(100vh - 120px);
        }
        .message-animation {
            animation: fadeIn 0.3s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .test-case-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }
        .typing-indicator span {
            animation: bounce 1.4s infinite ease-in-out;
        }
        .typing-indicator span:nth-child(2) {
            animation-delay: 0.2s;
        }
        .typing-indicator span:nth-child(3) {
            animation-delay: 0.4s;
        }
        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-5px); }
        }
    </style>
</head>
<body class="bg-gray-100 font-sans">
    <div class="container mx-auto max-w-6xl p-4">
        <!-- Header -->
        <header class="bg-white rounded-lg shadow-md p-4 mb-6 flex items-center justify-between">
            <div class="flex items-center">
                <div>
                    <h1 class="text-2xl font-bold text-gray-800">QE Assistant</h1>
                    <p class="text-sm text-gray-600">AI-powered test case generation and management</p>
                </div>
            </div>
            <div class="flex items-center space-x-4">
                <div class="relative group">
                    <button class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg flex items-center">
                        <i class="fas fa-plug mr-2"></i> Connect qTest
                    </button>
                    <div class="absolute right-0 mt-2 w-64 bg-white rounded-lg shadow-lg p-4 hidden group-hover:block z-10">
                        <p class="text-sm text-gray-700 mb-2">Connect to your qTest Manager instance:</p>
                        <input type="text" placeholder="qTest API URL" class="w-full p-2 border rounded mb-2 text-sm">
                        <input type="text" placeholder="API Token" class="w-full p-2 border rounded mb-2 text-sm">
                        <button class="w-full bg-green-600 text-white py-1 rounded text-sm">Save Connection</button>
                    </div>
                </div>
                <div class="bg-green-100 text-green-800 px-3 py-1 rounded-full text-sm flex items-center">
                    <span class="w-2 h-2 bg-green-500 rounded-full mr-2"></span>
                    Connected to Amazon Q
                </div>
            </div>
        </header>

        <!-- Main Content -->
        <div class="flex gap-6">
            <!-- Sidebar -->
            <div class="w-64 bg-white rounded-lg shadow-md p-4 hidden md:block">
                <h2 class="font-bold text-lg mb-4 text-gray-800">Test Projects</h2>
                <div class="space-y-2">
                    <div class="p-2 hover:bg-blue-50 rounded-lg cursor-pointer flex items-center">
                        <i class="fas fa-project-diagram text-blue-500 mr-2"></i>
                        <span>Web App Testing</span>
                    </div>
                    <div class="p-2 hover:bg-blue-50 rounded-lg cursor-pointer flex items-center">
                        <i class="fas fa-project-diagram text-blue-500 mr-2"></i>
                        <span>Mobile App Testing</span>
                    </div>
                    <div class="p-2 hover:bg-blue-50 rounded-lg cursor-pointer flex items-center">
                        <i class="fas fa-project-diagram text-blue-500 mr-2"></i>
                        <span>API Testing</span>
                    </div>
                </div>

                <h2 class="font-bold text-lg mt-6 mb-4 text-gray-800">Quick Actions</h2>
                <div class="space-y-2">
                    <button class="w-full text-left p-2 hover:bg-blue-50 rounded-lg cursor-pointer flex items-center" onclick="showTemplate('create-test-case')">
                        <i class="fas fa-plus-circle text-green-500 mr-2"></i>
                        <span>Create Test Case</span>
                    </button>
                    <button class="w-full text-left p-2 hover:bg-blue-50 rounded-lg cursor-pointer flex items-center" onclick="showTemplate('import-from-jira')">
                        <i class="fas fa-file-import text-purple-500 mr-2"></i>
                        <span>Import from JIRA</span>
                    </button>
                    <button class="w-full text-left p-2 hover:bg-blue-50 rounded-lg cursor-pointer flex items-center" onclick="showTemplate('generate-from-requirement')">
                        <i class="fas fa-magic text-yellow-500 mr-2"></i>
                        <span>Generate from Requirement</span>
                    </button>
                </div>

                <div class="mt-6 pt-4 border-t">
                    <h3 class="font-semibold text-gray-700 mb-2">Recent Test Cases</h3>
                    <div class="space-y-2">
                        <div class="text-sm p-2 hover:bg-gray-50 rounded cursor-pointer truncate">
                            <i class="fas fa-check-circle text-green-500 mr-2"></i>
                            TC-001: Login functionality
                        </div>
                        <div class="text-sm p-2 hover:bg-gray-50 rounded cursor-pointer truncate">
                            <i class="fas fa-check-circle text-green-500 mr-2"></i>
                            TC-002: Registration flow
                        </div>
                        <div class="text-sm p-2 hover:bg-gray-50 rounded cursor-pointer truncate">
                            <i class="fas fa-exclamation-circle text-yellow-500 mr-2"></i>
                            TC-003: Payment validation
                        </div>
                    </div>
                </div>
            </div>

            <!-- Chat Area -->
            <div class="flex-1 bg-white rounded-lg shadow-md overflow-hidden flex flex-col">
                <!-- Chat Header -->
                <div class="bg-gray-50 p-4 border-b flex items-center justify-between">
                    <div>
                        <h2 class="font-bold text-lg text-gray-800">Test Case Assistant</h2>
                        <p class="text-sm text-gray-600">Ask Amazon Q to help create, modify, or send test cases</p>
                    </div>
                    <div class="flex space-x-2">
                        <button class="p-2 hover:bg-gray-200 rounded-full" title="Clear Conversation">
                            <i class="fas fa-trash-alt text-gray-600"></i>
                        </button>
                        <button class="p-2 hover:bg-gray-200 rounded-full" title="Settings">
                            <i class="fas fa-cog text-gray-600"></i>
                        </button>
                    </div>
                </div>

                <!-- Chat Messages -->
                <div class="chat-container overflow-y-auto p-4 space-y-4" id="chat-messages">
                    <!-- Initial welcome message -->
                    <div class="message-animation flex">
                        <div class="w-8 h-8 rounded-full bg-blue-500 flex items-center justify-center text-white mr-3">
                            <i class="fas fa-robot"></i>
                        </div>
                        <div class="bg-blue-50 rounded-lg p-4 max-w-3xl">
                            <p class="text-gray-800">Hello! I'm Amazon Q, your testing assistant. I can help you:</p>
                            <ul class="list-disc pl-5 mt-2 space-y-1 text-gray-700">
                                <li>Create new test cases from requirements</li>
                                <li>Generate test data for your scenarios</li>
                                <li>Organize test cases into test cycles</li>
                                <li>Send test cases directly to qTest Manager</li>
                                <li>Analyze test coverage and suggest improvements</li>
                            </ul>
                            <p class="mt-3 text-gray-800">How can I help you with testing today?</p>
                        </div>
                    </div>

                    <!-- Example test case suggestion -->
                    <div class="message-animation flex justify-end">
                        <div class="bg-gray-100 rounded-lg p-4 max-w-3xl">
                            <p class="text-gray-800">Can you create a test case for user login functionality?</p>
                        </div>
                        <div class="w-8 h-8 rounded-full bg-gray-300 flex items-center justify-center text-white ml-3">
                            <i class="fas fa-user"></i>
                        </div>
                    </div>

                    <!-- Test case response -->
                    <div class="message-animation flex">
                        <div class="w-8 h-8 rounded-full bg-blue-500 flex items-center justify-center text-white mr-3">
                            <i class="fas fa-robot"></i>
                        </div>
                        <div class="bg-blue-50 rounded-lg p-4 max-w-3xl">
                            <p class="text-gray-800">Here's a test case for user login functionality:</p>
                            
                            <div class="mt-3 bg-white rounded-lg border p-4 test-case-card">
                                <div class="flex justify-between items-start mb-2">
                                    <h3 class="font-bold text-gray-800">TC-LOGIN-001: User Login Functionality</h3>
                                    <div class="flex space-x-2">
                                        <button class="text-blue-500 hover:text-blue-700" title="Edit">
                                            <i class="fas fa-edit"></i>
                                        </button>
                                        <button class="text-green-500 hover:text-green-700" title="Send to qTest">
                                            <i class="fas fa-paper-plane"></i>
                                        </button>
                                    </div>
                                </div>
                                <div class="text-sm space-y-3">
                                    <div>
                                        <p class="font-semibold text-gray-700">Description:</p>
                                        <p class="text-gray-600">Verify that a registered user can successfully log in to the application with valid credentials.</p>
                                    </div>
                                    <div>
                                        <p class="font-semibold text-gray-700">Preconditions:</p>
                                        <ul class="list-disc pl-5 text-gray-600">
                                            <li>User is registered in the system</li>
                                            <li>Application is loaded and accessible</li>
                                        </ul>
                                    </div>
                                    <div>
                                        <p class="font-semibold text-gray-700">Test Steps:</p>
                                        <ol class="list-decimal pl-5 text-gray-600">
                                            <li>Navigate to the login page</li>
                                            <li>Enter valid username in the username field</li>
                                            <li>Enter valid password in the password field</li>
                                            <li>Click the 'Login' button</li>
                                        </ol>
                                    </div>
                                    <div>
                                        <p class="font-semibold text-gray-700">Expected Results:</p>
                                        <ul class="list-disc pl-5 text-gray-600">
                                            <li>User is redirected to the dashboard page</li>
                                            <li>Welcome message with username is displayed</li>
                                            <li>Session is established with appropriate permissions</li>
                                        </ul>
                                    </div>
                                </div>
                                <div class="mt-3 pt-2 border-t flex justify-between items-center">
                                    <div class="flex space-x-2">
                                        <span class="bg-green-100 text-green-800 px-2 py-1 rounded-full text-xs">Functional</span>
                                        <span class="bg-blue-100 text-blue-800 px-2 py-1 rounded-full text-xs">Web</span>
                                    </div>
                                    <button class="text-xs text-gray-500 hover:text-gray-700">Add to Test Cycle</button>
                                </div>
                            </div>
                            
                            <div class="mt-3 flex space-x-3">
                                <button class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg text-sm">
                                    <i class="fas fa-paper-plane mr-2"></i> Send to qTest
                                </button>
                                <button class="bg-gray-200 hover:bg-gray-300 text-gray-800 px-4 py-2 rounded-lg text-sm">
                                    <i class="fas fa-edit mr-2"></i> Modify
                                </button>
                                <button class="bg-gray-200 hover:bg-gray-300 text-gray-800 px-4 py-2 rounded-lg text-sm">
                                    <i class="fas fa-plus mr-2"></i> Create Another
                                </button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Input Area -->
                <div class="p-4 border-t bg-gray-50">
                    <!-- Templates -->
                    <div class="mb-3 flex overflow-x-auto pb-2 space-x-2" id="templates-container">
                        <button class="flex-shrink-0 bg-white border border-blue-200 text-blue-600 px-3 py-1 rounded-full text-sm hover:bg-blue-50" onclick="insertTemplate('Create test case for login functionality')">
                            <i class="fas fa-sign-in-alt mr-1"></i> Login Test
                        </button>
                        <button class="flex-shrink-0 bg-white border border-blue-200 text-blue-600 px-3 py-1 rounded-full text-sm hover:bg-blue-50" onclick="insertTemplate('Generate test data for registration form')">
                            <i class="fas fa-user-plus mr-1"></i> Test Data
                        </button>
                        <button class="flex-shrink-0 bg-white border border-blue-200 text-blue-600 px-3 py-1 rounded-full text-sm hover:bg-blue-50" onclick="insertTemplate('Create a test cycle for regression testing')">
                            <i class="fas fa-recycle mr-1"></i> Test Cycle
                        </button>
                        <button class="flex-shrink-0 bg-white border border-blue-200 text-blue-600 px-3 py-1 rounded-full text-sm hover:bg-blue-50" onclick="insertTemplate('Send all new test cases to qTest Manager')">
                            <i class="fas fa-paper-plane mr-1"></i> Send to qTest
                        </button>
                    </div>
                    
                    <div class="flex items-center space-x-2">
                        <div class="flex-1 relative">
                            <textarea 
                                id="message-input" 
                                class="w-full p-3 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 pr-12 resize-none" 
                                rows="1" 
                                placeholder="Ask Amazon Q to help with testing..." 
                                oninput="autoResize(this)"></textarea>
                            <div class="absolute right-2 bottom-2 flex space-x-1">
                                <button class="p-2 text-gray-500 hover:text-blue-600" title="Attach File">
                                    <i class="fas fa-paperclip"></i>
                                </button>
                                <button class="p-2 text-gray-500 hover:text-blue-600" title="Insert Template">
                                    <i class="fas fa-magic"></i>
                                </button>
                            </div>
                        </div>
                        <button 
                            id="send-button"
                            class="bg-blue-600 hover:bg-blue-700 text-white p-3 rounded-lg flex items-center justify-center"
                            onclick="sendMessage()">
                            <i class="fas fa-paper-plane"></i>
                        </button>
                    </div>
                    <div class="mt-2 text-xs text-gray-500 flex justify-between">
                        <div>
                            <span>Amazon Q may produce inaccurate information. Always verify test cases.</span>
                        </div>
                        <div>
                            <button class="text-blue-600 hover:text-blue-800">Privacy</button>
                            <span class="mx-1">•</span>
                            <button class="text-blue-600 hover:text-blue-800">Terms</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Auto-resize textarea as user types
        function autoResize(textarea) {
            textarea.style.height = 'auto';
            textarea.style.height = (textarea.scrollHeight) + 'px';
        }

        // Insert template text into input
        function insertTemplate(text) {
            const input = document.getElementById('message-input');
            input.value = text;
            input.focus();
            autoResize(input);
        }

        // Show template options
        function showTemplate(type) {
            let message = '';
            switch(type) {
                case 'create-test-case':
                    message = 'Create a test case for [describe functionality] with [specific conditions]';
                    break;
                case 'import-from-jira':
                    message = 'Import test cases from JIRA ticket [ticket number]';
                    break;
                case 'generate-from-requirement':
                    message = 'Generate test cases from this requirement: [paste requirement]';
                    break;
            }
            insertTemplate(message);
        }

        // Send message to chat
        function sendMessage() {
            const input = document.getElementById('message-input');
            const message = input.value.trim();
            
            if (message === '') return;
            
            // Add user message to chat
            addMessageToChat(message, 'user');
            
            // Clear input
            input.value = '';
            autoResize(input);
            
            // Show typing indicator
            showTypingIndicator();
            
            // Simulate AI response after a delay
            setTimeout(() => {
                removeTypingIndicator();
                generateResponse(message);
            }, 1500);
        }

        // Add message to chat UI
        function addMessageToChat(message, sender) {
            const chatMessages = document.getElementById('chat-messages');
            
            const messageDiv = document.createElement('div');
            messageDiv.className = `message-animation flex ${sender === 'user' ? 'justify-end' : ''}`;
            
            if (sender === 'user') {
                messageDiv.innerHTML = `
                    <div class="bg-gray-100 rounded-lg p-4 max-w-3xl">
                        <p class="text-gray-800">${message}</p>
                    </div>
                    <div class="w-8 h-8 rounded-full bg-gray-300 flex items-center justify-center text-white ml-3">
                        <i class="fas fa-user"></i>
                    </div>
                `;
            } else {
                messageDiv.innerHTML = `
                    <div class="w-8 h-8 rounded-full bg-blue-500 flex items-center justify-center text-white mr-3">
                        <i class="fas fa-robot"></i>
                    </div>
                    <div class="bg-blue-50 rounded-lg p-4 max-w-3xl">
                        <p class="text-gray-800">${message}</p>
                    </div>
                `;
            }
            
            chatMessages.appendChild(messageDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        // Show typing indicator
        function showTypingIndicator() {
            const chatMessages = document.getElementById('chat-messages');
            
            const typingDiv = document.createElement('div');
            typingDiv.className = 'message-animation flex';
            typingDiv.id = 'typing-indicator';
            typingDiv.innerHTML = `
                <div class="w-8 h-8 rounded-full bg-blue-500 flex items-center justify-center text-white mr-3">
                    <i class="fas fa-robot"></i>
                </div>
                <div class="bg-blue-50 rounded-lg p-4 max-w-xs">
                    <div class="typing-indicator flex space-x-1">
                        <span class="w-2 h-2 bg-gray-500 rounded-full"></span>
                        <span class="w-2 h-2 bg-gray-500 rounded-full"></span>
                        <span class="w-2 h-2 bg-gray-500 rounded-full"></span>
                    </div>
                </div>
            `;
            
            chatMessages.appendChild(typingDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        // Remove typing indicator
        function removeTypingIndicator() {
            const typingIndicator = document.getElementById('typing-indicator');
            if (typingIndicator) {
                typingIndicator.remove();
            }
        }

        // Allow sending message with Enter key (but not Shift+Enter)
        document.getElementById('message-input').addEventListener('keydown', function(e) {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                sendMessage();
            }
        });
    </script>
</body>
</html>
