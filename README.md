# Webhacking-cookie.01
![image](https://user-images.githubusercontent.com/102034804/172055682-b3bfa67b-8437-4687-ac23-adb9e40ae345.png)
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

users = {
    'guest': 'guest',
    'user': 'user1234',
    'admin': FLAG
}


# this is our session storage 
session_storage = {
}


@app.route('/')
def index():
    session_id = request.cookies.get('sessionid', None)
    try:
        # get username from session_storage 
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html')

    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}')


@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    elif request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        try:
            # you cannot know admin's pw 
            pw = users[username]
        except:
            return '<script>alert("not found user");history.go(-1);</script>'
        if pw == password:
            resp = make_response(redirect(url_for('index')) )
            session_id = os.urandom(32).hex()
            session_storage[session_id] = username
            resp.set_cookie('sessionid', session_id)
            return resp 
        return '<script>alert("wrong password");history.go(-1);</script>'


@app.route('/admin')
def admin():
    # what is it? Does this page tell you session? 
    # It is weird... TODO: the developer should add a routine for checking privilege 
    return session_storage


if __name__ == '__main__':
    import os
    # create admin sessionid and save it to our storage
    # and also you cannot reveal admin's sesseionid by brute forcing!!! haha
    session_storage[os.urandom(32).hex()] = 'admin'
    print(session_storage)
    app.run(host='0.0.0.0', port=8000)



// /login 를 /admin 으로 수정한다-> app.route가 admin이기 때문!

![image](https://user-images.githubusercontent.com/102034804/172055747-9fe2f2fd-f42f-485e-aa8f-b96f60b075cd.png)

// /admin을 경로에 입력하면 해당 창이 나오게된다. 이때 쿠키의 세션 코드를 guest 또는 user 계정으로 로그인하여 이 계정들의 세션값에 저장하면 admin계정으로 로그인 성공!
![image](https://user-images.githubusercontent.com/102034804/172055820-91bf44b8-a012-4cbd-9eb8-9392cad1d26d.png)

//flag를 획득하게 된다.
