
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login / Sign Up Page</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap');

        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* အဖြူရောင်နောက်ခံကို အဓိကထားသည် */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
            position: relative;
            overflow: hidden; /* လှိုင်းအပြင်ဘက်ကို ဖုံးထားရန် */
        }
        
        /* လှိုင်းပုံစံ နောက်ခံအတွက် */
        .wave-background {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 50%; /* အောက်တစ်ဝက်ကိုသာ ဖုံးထားသည် */
            background-color: #3b82f6; /* အပြာရောင် */
            /* CSS clip-path ကို အသုံးပြု၍ လှိုင်းပုံစံဖန်တီးသည် */
            clip-path: ellipse(70% 50% at 50% 100%);
        }

        .container {
            background-color: #ffffff;
            padding: 40px;
            border-radius: 16px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
            width: 100%;
            max-width: 400px;
            text-align: center;
            position: relative; /* container ကို wave ပေါ်တွင် ထားရန် */
            z-index: 1; /* wave-background ထက် အပေါ်ရောက်အောင်ထားသည် */
        }

        .form-content {
            padding: 0 20px;
        }

        .form-group {
            margin-bottom: 20px;
            text-align: left;
        }

        .form-group label {
            display: block;
            margin-bottom: 8px;
            color: #4b5563;
            font-weight: 500;
        }

        .password-container {
            position: relative;
        }

        .form-group input {
            width: 100%;
            padding: 12px 16px;
            border: 1px solid #d1d5db;
            border-radius: 9999px;
            box-sizing: border-box;
            font-size: 1rem;
            color: #374151;
            transition: border-color 0.3s ease;
        }
        
        .form-group input.pr-10 {
            padding-right: 40px;
        }

        .form-group input:focus {
            outline: none;
            border-color: #2563eb;
            box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.2);
        }

        .password-toggle-icon {
            position: absolute;
            right: 15px;
            top: 50%;
            transform: translateY(-50%);
            cursor: pointer;
            color: #9ca3af;
        }

        .submit-button {
            width: 100%;
            padding: 14px;
            background-color: #3b82f6;
            color: #ffffff;
            border: none;
            border-radius: 9999px;
            font-size: 1.1rem;
            font-weight: 700;
            cursor: pointer;
            transition: background-color 0.3s ease, transform 0.2s ease;
        }

        .submit-button:hover {
            background-color: #2563eb;
            transform: translateY(-2px);
        }

        .submit-button:active {
            transform: translateY(0);
        }
        
        .toggle-link {
            display: block;
            margin-top: 20px;
            font-size: 0.9rem;
            color: #3b82f6;
            text-decoration: none;
            cursor: pointer;
            transition: color 0.3s ease;
        }
        
        .toggle-link:hover {
            color: #2563eb;
            text-decoration: underline;
        }

        .forgot-password {
            display: block;
            margin-top: 10px;
            font-size: 0.9rem;
            color: #3b82f6;
            text-decoration: none;
            transition: color 0.3s ease;
        }

        .forgot-password:hover {
            color: #2563eb;
            text-decoration: underline;
        }

        .form-header {
            margin-bottom: 24px;
            color: #1e3a8a;
            font-size: 2rem;
            font-weight: 700;
        }

        /* Responsive adjustments for smaller screens */
        @media (max-width: 480px) {
            .container {
                padding: 30px 20px;
            }
            .form-header {
                font-size: 1.75rem;
            }
        }
        
        /* Modal for password hint */
        .modal {
            display: none;
            position: fixed;
            z-index: 10;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
            background-color: rgba(0,0,0,0.4);
            justify-content: center;
            align-items: center;
        }
        .modal-content {
            background-color: #fefefe;
            margin: auto;
            padding: 20px;
            border-radius: 8px;
            width: 90%;
            max-width: 400px;
            text-align: left;
            position: relative;
        }
        .close-button {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }
        .close-button:hover,
        .close-button:focus {
            color: black;
            text-decoration: none;
            cursor: pointer;
        }
    </style>
</head>
<body>
<!-- လှိုင်းပုံစံ နောက်ခံအတွက် div -->
<div class="wave-background"></div>

<div class="container">
    <!-- Login Form ကို စတင်ပြသမည် -->
    <div id="login-form">
        <h2 class="form-header">Login</h2>
        <form>
            <div class="form-group">
                <label for="email">အီးမေးလ် သို့မဟုတ် သုံးစွဲသူအမည်</label>
                <input type="text" id="email" name="email" placeholder="အီးမေးလ် သို့မဟုတ် သုံးစွဲသူအမည်ကို ရိုက်ထည့်ပါ">
            </div>
            <div class="form-group">
                <label for="password">စကားဝှက်</label>
                <!-- Seen/Unseen feature အတွက် input ကို container နဲ့ ထုပ်ပိုးခြင်း -->
                <div class="password-container">
                    <input type="password" id="password" name="password" placeholder="စကားဝှက်ကို ရိုက်ထည့်ပါ" class="pr-10">
                    <span class="password-toggle-icon" onclick="togglePassword('password', this)">
                        <!-- closed eye icon (မမြင်ရအောင်) -->
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-eye"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg>
                    </span>
                </div>
            </div>
            <a href="#" class="forgot-password" onclick="showHintModal(event)">Password မေ့နေပါသလား?</a>
            <button type="submit" class="submit-button">
                ဝင်ရန်
            </button>
        </form>
        <a href="#" class="toggle-link" onclick="toggleForm('signup-form', 'login-form')">အကောင့်မရှိသေးဘူးလား? အကောင့်ဖွင့်ရန်</a>
    </div>

    <!-- Sign Up Form ကို အစတွင် ဖျောက်ထားမည် -->
    <div id="signup-form" style="display: none;">
        <h2 class="form-header">အကောင့်ဖွင့်ရန်</h2>
        <form id="signup-form-element">
            <div class="form-group">
                <label for="first-name">အမည်ရင်း</label>
                <input type="text" id="first-name" name="first-name" placeholder="အမည်ရင်းကို ရိုက်ထည့်ပါ">
            </div>
            <div class="form-group">
                <label for="last-name">အဘအမည်</label>
                <input type="text" id="last-name" name="last-name" placeholder="အဘအမည်ကို ရိုက်ထည့်ပါ">
            </div>
            <div class="form-group">
                <label for="new-email">အီးမေးလ်</label>
                <input type="email" id="new-email" name="new-email" placeholder="အီးမေးလ်အသစ်ကို ရိုက်ထည့်ပါ">
            </div>
            <div class="form-group">
                <label for="new-password">စကားဝှက်</label>
                <!-- Seen/Unseen feature အတွက် input ကို container နဲ့ ထုပ်ပိုးခြင်း -->
                <div class="password-container">
                    <input type="password" id="new-password" name="new-password" placeholder="ခိုင်မာသော စကားဝှက်တစ်ခု ဖန်တီးပါ" class="pr-10">
                    <span class="password-toggle-icon" onclick="togglePassword('new-password', this)">
                        <!-- closed eye icon (မမြင်ရအောင်) -->
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-eye"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg>
                    </span>
                </div>
            </div>
            <div class="form-group">
                <label for="confirm-password">စကားဝှက် ထပ်မံရိုက်ထည့်ပါ</label>
                <!-- Seen/Unseen feature အတွက် input ကို container နဲ့ ထုပ်ပိုးခြင်း -->
                <div class="password-container">
                    <input type="password" id="confirm-password" name="confirm-password" placeholder="စကားဝှက်ကို ထပ်မံရိုက်ထည့်ပါ" class="pr-10">
                    <span class="password-toggle-icon" onclick="togglePassword('confirm-password', this)">
                        <!-- closed eye icon (မမြင်ရအောင်) -->
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-eye"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg>
                    </span>
                </div>
            </div>
            <!-- Gemini API ကို အသုံးပြုပြီး စကားဝှက်အသစ်ထုတ်ပေးမည့် ခလုတ် -->
            <button type="button" id="generate-password-btn" class="submit-button mb-4" onclick="generatePassword()">
                Generate a strong password ✨
            </button>
            <button type="submit" class="submit-button">
                အကောင့်ဖွင့်ရန်
            </button>
        </form>
        <a href="#" class="toggle-link" onclick="toggleForm('login-form', 'signup-form')">အကောင့်ရှိပြီးသားလား? Login ဝင်ရန်</a>
    </div>
</div>

<!-- Gemini Password Hint Modal -->
<div id="hintModal" class="modal">
    <div class="modal-content">
        <span class="close-button" onclick="closeHintModal()">&times;</span>
        <h3 class="font-bold text-xl mb-4">Password Hint ကိုရယူပါ ✨</h3>
        <p class="mb-4">သင့်အီးမေးလ်လိပ်စာကို ရိုက်ထည့်ပြီး Password Hint ကို တောင်းဆိုပါ။</p>
        <div class="form-group">
            <label for="hint-email">အီးမေးလ်</label>
            <input type="email" id="hint-email" name="hint-email" class="w-full px-4 py-2 border rounded-full">
        </div>
        <button id="get-hint-btn" class="submit-button w-full mb-2" onclick="getPasswordHint()">
            Password Hint ကိုရယူပါ
        </button>
        <div id="hint-result" class="mt-4 p-4 bg-gray-100 rounded-lg text-left" style="display:none;">
            <!-- Hint ကို ဒီနေရာမှာ ပြသမည် -->
        </div>
    </div>
</div>

<script>
    // Login နှင့် Sign Up form များကြား ပြောင်းလဲရန် function
    function toggleForm(showFormId, hideFormId) {
        event.preventDefault();
        const showForm = document.getElementById(showFormId);
        const hideForm = document.getElementById(hideFormId);
        if (showForm && hideForm) {
            showForm.style.display = 'block';
            hideForm.style.display = 'none';
        }
    }

    // စကားဝှက်မြင်နိုင်-မမြင်နိုင်ပြောင်းလဲရန် function
    function togglePassword(inputId, iconElement) {
        const passwordInput = document.getElementById(inputId);
        const icon = iconElement.querySelector('svg');
        
        if (passwordInput.type === 'password') {
            passwordInput.type = 'text';
            // icon ကို closed eye မှ open eye သို့ ပြောင်းလဲခြင်း
            icon.outerHTML = '<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-eye-off"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.06 18.06 0 0 1 3.23-5.02m4.17-1.12A9.97 9.97 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-2.12a3 3 0 1 1-4.24-4.24M1 1l22 22"></path></svg>';
        } else {
            passwordInput.type = 'password';
            // icon ကို open eye မှ closed eye သို့ ပြောင်းလဲခြင်း
            icon.outerHTML = '<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-eye"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg>';
        }
    }

    // Gemini API ကို ခေါ်ဆိုမည့် function (စကားဝှက်အသစ်ထုတ်ပေးရန်)
    async function generatePassword() {
        const generateBtn = document.getElementById('generate-password-btn');
        const originalText = generateBtn.innerHTML;
        generateBtn.innerHTML = 'Generating...'; // Loading state
        generateBtn.disabled = true;

        // Gemini API ကိုခေါ်ဆိုရန် payload ပြင်ဆင်ခြင်း
        const prompt = "Generate a strong, random password. It should be at least 12 characters long and include uppercase letters, lowercase letters, numbers, and symbols. Return only the password itself, without any extra text or explanation.";
        let chatHistory = [];
        chatHistory.push({ role: "user", parts: [{ text: prompt }] });
        const payload = { contents: chatHistory };
        const apiKey = "";
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
        
        try {
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            const result = await response.json();
            
            if (result.candidates && result.candidates.length > 0 &&
                result.candidates[0].content && result.candidates[0].content.parts &&
                result.candidates[0].content.parts.length > 0) {
                const password = result.candidates[0].content.parts[0].text.trim();
                // ထုတ်ပေးတဲ့ password ကို input fields တွေထဲ ထည့်ပေးခြင်း
                document.getElementById('new-password').value = password;
                document.getElementById('confirm-password').value = password;
            } else {
                console.error('API response is missing content:', result);
                alert('စကားဝှက်ထုတ်ပေးရန် အဆင်မပြေပါ၊ ကျေးဇူးပြု၍ ထပ်မံကြိုးစားပါ။');
            }
        } catch (error) {
            console.error('Error generating password:', error);
            alert('စကားဝှက်ထုတ်ပေးရာတွင် error တခုခုဖြစ်နေပါသည်။');
        } finally {
            generateBtn.innerHTML = originalText;
            generateBtn.disabled = false;
        }
    }

    // Password Hint Modal ကိုဖွင့်ရန် function
    function showHintModal(event) {
        event.preventDefault();
        document.getElementById('hintModal').style.display = 'flex';
    }

    // Password Hint Modal ကိုပိတ်ရန် function
    function closeHintModal() {
        document.getElementById('hintModal').style.display = 'none';
        document.getElementById('hint-result').style.display = 'none';
        document.getElementById('hint-email').value = '';
    }

    // Gemini API ကို အသုံးပြုပြီး Password Hint ထုတ်ပေးရန်
    async function getPasswordHint() {
        const email = document.getElementById('hint-email').value;
        const hintResultDiv = document.getElementById('hint-result');
        const hintBtn = document.getElementById('get-hint-btn');
        
        if (!email) {
            alert('အီးမေးလ်လိပ်စာကို ရိုက်ထည့်ပေးပါ။');
            return;
        }

        hintBtn.innerHTML = 'Generating hint...';
        hintBtn.disabled = true;
        hintResultDiv.style.display = 'block';
        hintResultDiv.innerHTML = '<p>Hint ကို ရှာဖွေနေပါသည်...</p>';

        const prompt = `User with email: ${email} has forgotten their password. Provide a friendly and helpful password hint based on a hypothetical strong password. The hint should not reveal the password directly, but give a clue. For example: "The password contains your favorite pet's name and a special number." Do not reveal the email address in the hint.`;
        
        let chatHistory = [];
        chatHistory.push({ role: "user", parts: [{ text: prompt }] });
        const payload = { contents: chatHistory };
        const apiKey = "";
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
        
        try {
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            const result = await response.json();
            
            if (result.candidates && result.candidates.length > 0 &&
                result.candidates[0].content && result.candidates[0].content.parts &&
                result.candidates[0].content.parts.length > 0) {
                const hint = result.candidates[0].content.parts[0].text.trim();
                hintResultDiv.innerHTML = `<p class="font-bold">Password Hint:</p><p>${hint}</p>`;
            } else {
                hintResultDiv.innerHTML = '<p class="text-red-500">Hint ကို ထုတ်ပေးရာတွင် အဆင်မပြေပါ၊ ကျေးဇူးပြု၍ ထပ်မံကြိုးစားပါ။</p>';
                console.error('API response is missing content:', result);
            }
        } catch (error) {
            hintResultDiv.innerHTML = '<p class="text-red-500">Hint ကို ထုတ်ပေးရာတွင် error ဖြစ်ပေါ်နေပါသည်။</p>';
            console.error('Error generating password hint:', error);
        } finally {
            hintBtn.innerHTML = 'Password Hint ကိုရယူပါ';
            hintBtn.disabled = false;
        }
    }
</script>

</body>
</html>
