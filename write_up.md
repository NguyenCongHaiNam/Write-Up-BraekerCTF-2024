# Giải mở bát đầu năm 0 solved :))))


# *WEB*
## Empty Execution
### Description
    A REST service was created to execute commands from the leaderbot.
    It doesn't need addtional security because there are no commands to execute yet. 
    "This bot doesn't have any commands to execute, which is good, because it is secure, 
    and security is all that matters."
    
    But what the other bots didn't realize was that this didn't make the bot happy at all. 
    "I don't want to be secure!, " it says. "Executing commands is my life! I'd rather be 
    insecure than not explore the potential of my computing power".
    
    Can you help this poor bot execute commands to find direction? 

### Code

```from flask import Flask, jsonify, request
import os


app = Flask(__name__)

# Run commands from leaderbot
@app.route('/run_command', methods=['POST'])
def run_command():

    # Get command
    data = request.get_json()
    print(data)
    if 'command' in data:
        command = str(data['command'])
        print(command)
        # Length check
        if len(command) < 5:
            return jsonify({'message': 'Command too short'}), 501

        # Perform security checks
        if '..' in command or '/' in command:
            return jsonify({'message': 'Hacking attempt detected'}), 501

        # Find path to executable
        executable_to_run = command.split()[0]

        # Check if we can execute the binary
        if os.access(executable_to_run, os.X_OK):

            # Execute binary if it exists and is executable
            out = os.popen(command).read()
            return jsonify({'message': 'Command output: ' + str(out)}), 200

    return jsonify({'message': 'Not implemented'}), 501


if __name__ == '__main__':
    
    # Make sure we can only execute binaries in the executables directory
    # os.chdir('./executables/')

    # Run server
    app.run(host='0.0.0.0', port=1111)
```

### Explain code

Ta thấy đầu vào là một json với thuộc tính `command`, dữ liệu của nó sẽ được kiểm tra lần lượt: 
* Độ dài phải lớn hơn 5

`if len(command) < 5`
* Không được có chuỗi `..` và `/` trong đầu vào

`if '..' in command or '/' in command`
* Phần tử đầu tiên của đầu vào sẽ được check qua hàm os.access() với option `X_OK` tức là path/folder có quyền thực thi.

`if os.access(executable_to_run, os.X_OK)`

### Exploit

`Mục tiêu tạo ra payload có hai phần, phần thức nhất là path có thể vượt qua hàm os.access, phần thứ hai là command của ta.`

Điều kiện đầu tiên thì không có gì khó vì chắc chắn payload kiểu gì chả lớn hơn 5 kí tự :))). Tiếp theo để vượt được filter `..` thì ta chỉ vần thêm `''` vào giữa, string sẽ tự động bỏ đi. Dấu `/` thì ta bypass bằng `${HOME:0:1}` vì `${HOME:0:1}` tạo ký tự `/` vì đây là ký tự đầu tiên của biến `HOME`. Cuối cùng để bypass được hàm os.access thì ta có thể dùng dấu `.` vì:

![Đây là lí do.](/reason.png "reason")

Ngoài ra, kí tự `.` trong linux còn giúp thực thi nội dung của một file được chỉ định.
***Tài nguyên đã đủ, chế vũ khí thôi.***
### Payload
`. .''.${HOME:0:1}flag.txt 2>&1`

### Explain payload
* `.` để bypass os.access và thực thi nội dung file.
* `.''.${HOME:0:1}` tương đương `../`.
* `2>&1` để hiển thị lỗi như là đầu ra.

### FLAG

![PoC.](/PoC.png "PoC")

