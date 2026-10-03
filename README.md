# ai-recipe-generator
4
<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI রেসিপি ডিটেক্টর (Food Recipe AI)</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
        }
        body {
            background-color: #f4f7f6;
            color: #333;
            display: flex;
            justify-content: center;
            padding: 20px;
        }
        .container {
            max-width: 650px;
            width: 100%;
            background: #ffffff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
        }
        h1 {
            text-align: center;
            color: #ff5722;
            margin-bottom: 20px;
        }
        .upload-section {
            border: 2px dashed #ff5722;
            padding: 20px;
            text-align: center;
            border-radius: 8px;
            background-color: #fff8f6;
            margin-bottom: 20px;
        }
        input[type="file"] {
            display: none;
        }
        .custom-file-upload {
            display: inline-block;
            padding: 10px 20px;
            cursor: pointer;
            background-color: #ff5722;
            color: white;
            border-radius: 5px;
            font-weight: bold;
            transition: 0.3s;
        }
        .custom-file-upload:hover {
            background-color: #e64a19;
        }
        #preview {
            max-width: 100%;
            max-height: 250px;
            margin-top: 15px;
            border-radius: 8px;
            display: none;
        }
        button {
            width: 100%;
            padding: 12px;
            background-color: #2e7d32;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 16px;
            cursor: pointer;
            font-weight: bold;
            margin-top: 10px;
        }
        button:hover {
            background-color: #1b5e20;
        }
        #loader {
            display: none;
            text-align: center;
            margin-top: 15px;
            font-weight: bold;
            color: #ff5722;
        }
        .result-box {
            margin-top: 20px;
            padding: 20px;
            background-color: #f9f9f9;
            border-left: 5px solid #2e7d32;
            border-radius: 5px;
            white-space: pre-line;
            line-height: 1.6;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>🍳 AI রেসিপি ডিটেক্টর</h1>
    <p style="text-align: center; margin-bottom: 15px;">আপনার রান্নার যেকোনো ছবি দিন, এআই বানিয়ে দেবে রেসিপি!</p>
    
    <div class="upload-section">
        <label for="imageInput" class="custom-file-upload">
            📸 রান্নার ছবি সিলেক্ট করুন
        </label>
        <input type="file" id="imageInput" accept="image/*" onchange="previewImage(event)">
        <br>
        <img id="preview" alt="Image Preview">
    </div>

    <button onclick="analyzeFoodImage()">রেসিপি তৈরি করো</button>

    <div id="loader">⏳ ছবিটি পর্যবেক্ষণ করা হচ্ছে... অনুগ্রহ করে অপেক্ষা করুন।</div>

    <div id="result" class="result-box" style="display: none;"></div>
</div>

<script>
    let base64Image = "";

    // ছবি প্রিভিউ দেখানোর ফাংশন
    function previewImage(event) {
        const file = event.target.files[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = function(e) {
                const preview = document.getElementById('preview');
                preview.src = e.target.result;
                preview.style.display = 'block';
                base64Image = e.target.result.split(',')[1]; // Extract Base64 string
            };
            reader.readAsDataURL(file);
        }
    }

    // AI দিয়ে ছবি বিশ্লেষণ করার ফাংশন
    async function analyzeFoodImage() {
        if (!base64Image) {
            alert("অনুগ্রহ করে একটি রান্নার ছবি নির্বাচন করুন!");
            return;
        }

        const loader = document.getElementById('loader');
        const resultDiv = document.getElementById('result');
        
        loader.style.display = 'block';
        resultDiv.style.display = 'none';

        // ⚠️ আপনার Google Gemini API Key নিচে বসান 
        const API_KEY = "YOUR_GEMINI_API_KEY"; 

        const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${API_KEY}`;

        const promptText = "এই ছবির খাবারটি চিহ্নিত করো এবং এটি কীভাবে তৈরি করা হয়েছে তার সম্পূর্ণ রেসিপি পরিষ্কার বাংলায় বিস্তারিত ধাপে ধাপে লিখে দাও। প্রথমে খাবারের নাম, প্রয়োজনীয় উপকরণ এবং তারপর তৈরির পদ্ধতি দাও।";

        const requestBody = {
            contents: [{
                parts: [
                    { text: promptText },
                    {
                        inline_data: {
                            mime_type: "image/jpeg",
                            data: base64Image
                        }
                    }
                ]
            }]
        };

        try {
            const response = await fetch(url, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify(requestBody)
            });

            const data = await response.json();
            loader.style.display = 'none';

            if (data.candidates && data.candidates[0].content.parts[0].text) {
                resultDiv.innerText = data.candidates[0].content.parts[0].text;
                resultDiv.style.display = 'block';
            } else {
                resultDiv.innerText = "দুঃখিত, খাবারটি চিহ্নিত করা সম্ভব হয়নি। আবার চেষ্টা করুন।";
                resultDiv.style.display = 'block';
            }
        } catch (error) {
            loader.style.display = 'none';
            alert("কিছু সমস্যা হয়েছে! আবার চেষ্টা করুন।");
            console.error(error);
        }
    }
</script>

</body>
</html>
