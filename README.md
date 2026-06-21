<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Rider</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        * {
            box-sizing: border-box;
        }
        body {
            margin: 0;
            padding: 0;
            background: #070810;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            color: #ffffff;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        .header-title {
            margin: 10px 0;
            font-size: 1.6rem;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 5px;
            color: #00fff2;
            text-shadow: 0 0 10px rgba(0, 255, 242, 0.3);
            text-align: center;
        }
        .container {
            position: relative;
            border-radius: 16px;
            overflow: hidden;
            width: 92%;
            max-width: 450px;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.7);
            border: 2px solid #00fff2;
            background: #090a14;
        }
        canvas {
            display: block;
            width: 100%;
            height: auto;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 55%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 14px 32px;
            font-size: 1rem;
            font-weight: 600;
            letter-spacing: 1px;
            background: #00fff2;
            color: #000000;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 0 20px rgba(0, 255, 242, 0.5);
            z-index: 10;
        }
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 92%;
            max-width: 450px;
            margin-top: 25px;
            margin-bottom: 20px;
            padding: 0 10px;
        }
        .btn-group {
            display: flex;
            gap: 16px;
        }
        .btn {
            width: 65px;
            height: 65px;
            background: rgba(9, 10, 20, 0.8);
            border: 2px solid #00fff2;
            border-radius: 16px;
            color: #00fff2;
            font-size: 1.5rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
            box-shadow: 0 4px 10px rgba(0, 255, 242, 0.1);
        }
        .btn:active {
            background: #00fff2;
            color: #090a14;
            box-shadow: 0 0 15px rgba(0, 255, 242, 0.6);
        }
        .btn-action {
            color: #ff2d55;
            border-color: #ff2d55;
            box-shadow: 0 4px 10px rgba(255, 45, 85, 0.1);
        }
        .btn-action:active {
            background: #ff2d55;
            color: #090a14;
            box-shadow
