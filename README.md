# webappcam

##index.html

기존의 https://github.com/ouseok00/webApp.git 을 확장해보려고 한다.

1.웹브라우저에서 파일을 선택해서 서버로 전송한다.
2. 서버는 이미지 파일을 /static/uploads 폴더에 저장한 후

3. addbook.txt 파일에 이미지 이름을 저장하게 한다.
![image](https://github.com/user-attachments/assets/2a5508db-5c6a-4f8d-ac9d-424e3fe04874)
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

##app.py

```
from flask import Flask, render_template, request, redirect, url_for
import csv
import os

app = Flask(__name__)

## 폴더 설정
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

 # 사진 저장
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

