# webappcam

기존의 https://github.com/ouseok00/webApp.git 을 확장해보려고 한다.

1.웹브라우저에서 파일을 선택해서 서버로 전송한다.

2.서버는 이미지 파일을 /static/uploads 폴더에 저장한 후

3.addbook.txt 파일에 이미지 이름을 저장하게 한다.
![image](https://github.com/user-attachments/assets/2a5508db-5c6a-4f8d-ac9d-424e3fe04874)

# 주요 수정 내용

## 1. index.html
```
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
    <title>Address Book</title>
</head>
<body>
    <h1>Address Book</h1>
    <form action="/add" method="post" enctype="multipart/form-data">
        <label for="name" class="name">이름:</label>
        <input type="text" id="name" name="name" required>
        <br><br>
        <label for="phone">전화번호:</label>
        <input type="text" id="phone" name="phone" required>
        <br><br>
        <label for="birthday">생일:</label>
        <input type="text" id="birthday" name="birthday" required>
        <br><br>
        <label for="photo">사진 첨부:</label>
        <input type="file" id="photo" name="photo" accept="image/*">
        <br><br>
        <button type="submit">전송</button>
    </form>
</body>
</html>
```

action="/add" 폼을 제출하면 데이터를 /add 경로(Flask의 /add라우터)로 보냅니다.
method="post" 데이터를 서버로 보낼 때 HTTP POST 방식(숨겨진 방식, URL에 데이터가 노출되지 않음)으로 전송합니다.
enctype="multipart/form-data" 이 속성이 있어야 파일 데이터가 서버로 전송됩니다.

## 2. app.py

```
from flask import Flask, render_template, request, redirect, url_for
import csv
import os

app = Flask(__name__)

UPLOAD_FOLDER = 'static/uploads'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/add', methods=['POST'])
def add_contact():
    name = request.form['name']
    phone = request.form['phone']
    birthday = request.form['birthday']
    photo = request.files['photo']
    if photo:
        photo_path = os.path.join(app.config['UPLOAD_FOLDER'], photo.filename)
        photo.save(photo_path)

    # Save to addbook.txt in CSV format
    with open('addbook.txt', 'a', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow([name, phone, birthday, photo.filename if photo else ''])

    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
```
파일 내용을 받아서 static/uploads 폴더에 저장하고 파일이름을 addbook.txt에 함께 저장한다.

## 실행화면
![image](https://github.com/user-attachments/assets/e8885c88-cc2e-4abd-ba4b-e1305504c7c2)

uploads 폴더에 이미지가 저장된 모습

![image](https://github.com/user-attachments/assets/97c2053b-78e6-4fbc-ae7d-acc13090bc4d)

## 웹카메라로 직접 촬영해서 올리기

#1. index.html에서 

버튼 추가

![image](https://github.com/user-attachments/assets/8cfaac1e-a1b7-42d9-a8ba-ae6bd1c103e8)

함수 추가

![image](https://github.com/user-attachments/assets/09c9b33d-0b88-46f7-87eb-9c77f45525b8)

# 전체코드
```
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
    <title>Address Book</title>
</head>
<body>
    <h1>Address Book</h1>
    <form action="/add" method="post" enctype="multipart/form-data">
        <label for="name" class="name">이름:</label>
        <input type="text" id="name" name="name" required>
        <br><br>
        <label for="phone">전화번호:</label>
        <input type="text" id="phone" name="phone" required>
        <br><br>
        <label for="birthday">생일:</label>
        <input type="text" id="birthday" name="birthday" required>
        <br><br>
        <label for="photo">사진 첨부:</label>
        <input type="file" id="photo" name="photo" accept="image/*" onchange="previewFile()">
        <button type="button" onclick="openCamera()">웹캠 촬영</button>
        <br><br>
        <video id="camera" width="200" autoplay style="display:none;"></video>
        <canvas id="canvas" width="200" style="display:none;"></canvas>
        <!-- 미리보기 이미지 -->
        <img id="preview" src="#" alt="이미지 미리보기" style="display: none; max-width: 200px; margin-top: 10px;">
        <!-- 웹캠 촬영 미리보기 이미지 -->
        <img id="photoPreview" src="#" alt="웹캠 이미지 미리보기" style="display: none; max-width: 200px; margin-top: 10px;">
        <br><br>
        <button type="submit">전송</button>
    </form>

    <script>
        function previewFile() {
            const preview = document.getElementById('preview');
            const file = document.getElementById('photo').files[0];
            const reader = new FileReader();

            if (file) {
                reader.onload = function(event) {
                    preview.src = event.target.result;
                    preview.style.display = 'block';
                };
                reader.readAsDataURL(file);
            } else {
                preview.src = '#';
                preview.style.display = 'none';
            }
        }

        function openCamera() {
            const video = document.getElementById('camera');
            const canvas = document.getElementById('canvas');
            video.style.display = 'block';
            navigator.mediaDevices.getUserMedia({ video: true })
                .then(stream => {
                    video.srcObject = stream;
                    video.play();
                    if (!document.getElementById('captureBtn')) {
                        const btn = document.createElement('button');
                        btn.textContent = '촬영';
                        btn.type = 'button';
                        btn.id = 'captureBtn';
                        btn.onclick = function() {
                            capturePhoto(video, canvas);
                        };
                        video.parentNode.insertBefore(btn, video.nextSibling);
                    }
                })
                .catch(err => {
                    alert('웹캠을 사용할 수 없습니다: ' + err);
                });
        }

        function capturePhoto(video, canvas) {
            const context = canvas.getContext('2d');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            const dataURL = canvas.toDataURL('image/png');
            document.getElementById('photoPreview').src = dataURL;
            document.getElementById('photoPreview').style.display = 'block';
            document.getElementById('photo').value = '';
            let hidden = document.getElementById('webcamImage');
            if (!hidden) {
                hidden = document.createElement('input');
                hidden.type = 'hidden';
                hidden.name = 'webcamImage';
                hidden.id = 'webcamImage';
                document.querySelector('form').appendChild(hidden);
            }
            hidden.value = dataURL;
            if (video.srcObject) {
                video.srcObject.getTracks().forEach(track => track.stop());
            }
            video.style.display = 'none';
            canvas.style.display = 'none';
            const btn = document.getElementById('captureBtn');
            if (btn) btn.remove();
        }
    </script>
</body>
</html>
```

## 2.app.py

파일 업로드와 웹카메라를 선택적으로 저장
![image](https://github.com/user-attachments/assets/92ba2842-9d39-49a5-bed3-3754c654af9c)

```
from flask import Flask, render_template, request, redirect, url_for
import csv
import os
import base64
from datetime import datetime

app = Flask(__name__)

UPLOAD_FOLDER = 'static/uploads'
os.makedirs(UPLOAD_FOLDER, exist_ok=True)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/add', methods=['POST'])
def add_contact():
    name = request.form.get('name')
    phone = request.form.get('phone')
    birthday = request.form.get('birthday')
    photo = request.files.get('photo')
    webcam_image = request.form.get('webcamImage')

    photo_name_to_save = ''
    if photo and photo.filename:
        photo_path = os.path.join(app.config['UPLOAD_FOLDER'], photo.filename)
        photo.save(photo_path)
        photo_name_to_save = photo.filename
    elif webcam_image:
        now_str = datetime.now().strftime('%Y%m%d_%H%M%S')
        photo_name_to_save = f'webcam_{now_str}.png'
        photo_path = os.path.join(app.config['UPLOAD_FOLDER'], photo_name_to_save)
        header, data = webcam_image.split(',', 1)
        with open(photo_path, 'wb') as f:
            f.write(base64.b64decode(data))

    with open('addbook.txt', 'a', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow([name, phone, birthday, photo_name_to_save])

    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
```

## 실행화면
![image](https://github.com/user-attachments/assets/f7479cda-6299-4a86-8c65-df07c320fa35)
