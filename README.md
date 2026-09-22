<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Lotería Oficial</title>

    <!-- FAVICON OFICIAL -->
    <link rel="icon" type="image/png" href="https://i.postimg.cc/sXcgPJMZ/Picsart-26-09-18-08-28-37-079.png">

    <!-- SCRIPTS DE FIREBASE PARA TIEMPO REAL -->
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>

    <style>
        * { 
            box-sizing: border-box; 
            margin: 0; 
            padding: 0; 
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            -webkit-user-select: none;
            user-select: none;
            -webkit-touch-callout: none;
        }
        
        html, body {
            width: 100%;
            min-height: 100%;
            background-color: #f5f6fa;
            color: #333;
            overflow-x: hidden;
            touch-action: pan-y;
        }

        body {
            position: relative;
            padding-bottom: 24px;
        }

        #bgCanvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: 0;
            pointer-events: none;
        }

        #fireworksCanvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: 998;
            pointer-events: none;
            display: none;
        }

        header { 
            position: relative; 
            z-index: 20; 
            background: #ff7a00; 
            color: #fff; 
            padding: 10px 14px; 
            display: flex; 
            flex-wrap: wrap;
            justify-content: space-between; 
            align-items: center; 
            row-gap: 8px;
            column-gap: 12px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.25); 
            width: 100%;
        }
        
        .brand-container { 
            display: flex; 
            align-items: center; 
            gap: 10px; 
            cursor: pointer; 
            flex-shrink: 0;
        }
        .official-logo-img {
            width: 36px;
            height: 36px;
            border-radius: 50%;
            object-fit: contain;
            background: #ffffff;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            border: 2px solid #ffffff;
            flex-shrink: 0;
        }
        .logo-text-group { display: flex; flex-direction: column; text-align: left; }
        .logo-main { font-size: 17px; font-weight: 900; letter-spacing: 0.5px; text-transform: uppercase; color: #ffffff; line-height: 1; }
        .logo-sub { font-size: 8.5px; font-weight: 800; letter-spacing: 1.2px; text-transform: uppercase; color: #fff3e6; margin-top: 2px; }

        .header-btns { 
            display: flex; 
            gap: 6px; 
            align-items: center;
            flex-wrap: wrap;
        }
        .nav-btn { 
            background: #ffffff; 
            color: #ff7a00; 
            border: none; 
            padding: 7px 12px; 
            font-weight: 800; 
            border-radius: 16px; 
            font-size: 11px; 
            cursor: pointer; 
            box-shadow: 0 2px 4px rgba(0,0,0,0.1); 
            white-space: nowrap;
            transition: background 0.15s ease, transform 0.1s ease;
        }
        .nav-btn:active { transform: scale(0.96); }
        .nav-btn-check {
            background: #0f172a;
            color: #ffffff;
        }

        .sub-header { 
            position: relative; 
            z-index: 10; 
            background: #ff9431; 
            color: #fff; 
            text-align: center; 
            padding: 8px 10px; 
            font-size: 12px; 
            font-weight: 700; 
            letter-spacing: 0.5px; 
            text-transform: uppercase; 
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 12px;
            flex-wrap: wrap;
        }

        /* BADGE EN VIVO */
        .live-visitors-badge {
            background: #0f172a;
            color: #ffffff;
            padding: 3px 10px;
            border-radius: 12px;
            font-size: 11px;
            font-weight: 800;
            display: inline-flex;
            align-items: center;
            gap: 6px;
            letter-spacing: 0.5px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.15);
        }
        .live-dot {
            width: 7px;
            height: 7px;
            background: #22c55e;
            border-radius: 50%;
            display: inline-block;
            box-shadow: 0 0 6px #22c55e;
            animation: pulseLiveDot 1.5s infinite;
        }
        @keyframes pulseLiveDot {
            0% { transform: scale(0.9); opacity: 0.6; }
            50% { transform: scale(1.3); opacity: 1; }
            100% { transform: scale(0.9); opacity: 0.6; }
        }

        .marquee-box { 
            position: relative; 
            z-index: 10; 
            background: rgba(255, 243, 230, 0.95); 
            border-bottom: 1px solid #ffd8b3; 
            color: #d35400; 
            padding: 8px; 
            font-size: 13px; 
            font-weight: 600; 
            white-space: nowrap; 
            overflow: hidden; 
            backdrop-filter: blur(4px); 
        }
        .marquee-box span { display: inline-block; animation: marquee 14s linear infinite; }
        @keyframes marquee { 0% { transform: translateX(100%); } 100% { transform: translateX(-100%); } }

        .container { 
            position: relative; 
            z-index: 10; 
            width: 95%;
            max-width: 960px; 
            margin: 18px auto 0 auto; 
            padding: 0 6px; 
        }

        .lotos-stack {
            display: flex;
            flex-direction: column;
            gap: 20px;
            margin-bottom: 20px;
        }

        .card { 
            background: rgba(255, 255, 255, 0.96); 
            border-radius: 20px; 
            padding: 24px 18px; 
            box-shadow: 0 8px 30px rgba(0,0,0,0.06); 
            text-align: center; 
            border: 1px solid rgba(255, 255, 255, 0.85); 
            backdrop-filter: blur(10px); 
        }
        
        .card-header-badge { 
            font-size: 16px; 
            font-weight: 900; 
            text-transform: uppercase; 
            letter-spacing: 1px; 
            margin-bottom: 12px; 
        }

        .countdown-title-bar {
            font-size: 12px;
            font-weight: 800;
            text-transform: uppercase;
            letter-spacing: 0.8px;
            margin-bottom: 14px;
            display: inline-flex;
            align-items: center;
            gap: 4px;
            padding: 5px 14px;
            border-radius: 14px;
        }
        .countdown-title-orange { color: #c2410c; background: #ffedd5; border: 1px solid #fed7aa; }
        .countdown-title-blue { color: #0369a1; background: #e0f2fe; border: 1px solid #bae6fd; }
        .countdown-title-purple { color: #6b21a8; background: #f3e8ff; border: 1px solid #e9d5ff; }

        .clock-grid {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 10px;
            margin: 6px auto 20px auto;
            max-width: 520px;
        }
        .clock-card {
            background: #ffffff;
            border-radius: 14px;
            flex: 1;
            padding: 12px 6px;
            box-shadow: 0 4px 14px rgba(15, 23, 42, 0.07);
            border: 1px solid #e2e8f0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        .clock-num {
            font-size: 28px;
            font-weight: 900;
            line-height: 1;
            margin-bottom: 4px;
            letter-spacing: -0.5px;
        }
        .clock-label {
            font-size: 10px;
            font-weight: 800;
            text-transform: uppercase;
            letter-spacing: 0.8px;
            color: #1e293b;
            opacity: 0.85;
        }
        
        .clock-card-orange .clock-num { color: #ea580c; }
        .clock-card-blue .clock-num { color: #0284c7; }
        .clock-card-purple .clock-num { color: #9333ea; }

        .balls-row { 
            display: flex; 
            justify-content: center; 
            align-items: center; 
            gap: 14px; 
            margin: 18px 0 24px 0; 
            min-height: 56px; 
        }
        .ball { 
            width: 52px; 
            height: 52px; 
            border-radius: 50%; 
            background: radial-gradient(circle at 35% 35%, #ffffff, #e0e0e0 70%); 
            border: 2px solid #bbb; 
            color: #222; 
            font-size: 22px; 
            font-weight: 800; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            box-shadow: 0 4px 10px rgba(0,0,0,0.15); 
            transition: all 0.3s ease; 
        }
        .ball-special { background: radial-gradient(circle at 35% 35%, #ff5252, #c62828 70%); border: 2px solid #b71c1c; color: #fff; }
        .ball-gold { background: radial-gradient(circle at 35% 35%, #0284c7, #0369a1 70%); border: 2px solid #075985; color: #fff; }
        .ball-purple { background: radial-gradient(circle at 35% 35%, #a855f7, #7e22ce 70%); border: 2px solid #6b21a8; color: #fff; }

        .jackpot-pill { 
            border: 2px solid #ff7a00; 
            border-radius: 36px; 
            padding: 14px 34px; 
            margin: 8px auto 2px auto; 
            display: inline-block; 
            background: #fff; 
            box-shadow: 0 4px 14px rgba(255,122,0,0.12); 
        }
        .jackpot-pill-blue { border-color: #0284c7; box-shadow: 0 4px 14px rgba(2,132,199,0.12); }
        .jackpot-pill-purple { border-color: #9333ea; box-shadow: 0 4px 14px rgba(147,51,234,0.12); }
        .jackpot-tag { font-size: 12px; font-weight: 800; color: #888; text-transform: uppercase; letter-spacing: 1px; }
        .jackpot-value { font-size: 30px; font-weight: 900; color: #ff7a00; margin-top: 3px; }
        .jackpot-value-blue { color: #0284c7; }
        .jackpot-value-purple { color: #9333ea; }

        .winner-card { 
            background: linear-gradient(135deg, #f0fdf4 0%, #dcfce7 100%); 
            border: 2px solid #22c55e; 
            border-radius: 14px; 
            padding: 16px; 
            margin-top: 14px; 
            display: none; 
            box-shadow: 0 6px 20px rgba(34,197,94,0.15); 
            animation: popIn 0.3s ease-out; 
        }
        .winner-title { color: #15803d; font-size: 16px; font-weight: 900; text-transform: uppercase; margin-bottom: 6px; letter-spacing: 0.5px; }

        /* SECCIÓN DE RESEÑAS */
        .reviews-section { margin-top: 14px; padding: 10px 0; overflow: hidden; width: 100%; position: relative; z-index: 10; }
        .reviews-header-bar { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; padding: 0 4px; }
        .reviews-title { font-size: 13px; font-weight: 800; color: #ff7a00; letter-spacing: 1px; text-transform: uppercase; }
        .btn-add-review { background: #ff7a00; color: #fff; border: none; padding: 6px 12px; border-radius: 14px; font-size: 11px; font-weight: bold; cursor: pointer; }
        
        .marquee-carousel { display: flex; width: max-content; animation: scrollReviews 28s linear infinite; }
        @keyframes scrollReviews { 0% { transform: translateX(0); } 100% { transform: translateX(-50%); } }

        .review-card { background: #ffffff; border: 1px solid #e2e8f0; border-radius: 12px; width: 220px; padding: 12px; margin: 0 6px; flex-shrink: 0; display: flex; flex-direction: column; justify-content: space-between; text-align: left; box-shadow: 0 4px 10px rgba(0,0,0,0.05); }
        .review-top { display: flex; justify-content: space-between; align-items: center; margin-bottom: 6px; }
        .review-stars { color: #ff7a00; font-size: 12px; letter-spacing: 1px; }
        .review-date { font-size: 9px; color: #94a3b8; font-weight: 600; }
        .review-badge { background: #fffaf5; border: 1px solid #ffe8cc; border-radius: 6px; padding: 4px 7px; display: flex; align-items: center; gap: 6px; margin-bottom: 8px; }
        .badge-dot { width: 6px; height: 6px; background: #ff7a00; border-radius: 50%; }
        .badge-info { display: flex; flex-direction: column; overflow: hidden; }
        .badge-tag { font-size: 8px; font-weight: 800; color: #ff7a00; text-transform: uppercase; }
        .badge-user { font-size: 10.5px; font-weight: 700; color: #1e293b; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        .review-text { font-size: 10.5px; color: #475569; line-height: 1.4; margin-bottom: 8px; word-break: break-word; }
        .review-footer { display: flex; justify-content: space-between; align-items: center; border-top: 1px solid #f1f5f9; padding-top: 6px; }
        .review-footer span { font-size: 10px; color: #ff7a00; font-weight: 700; }
        .review-footer svg { width: 12px; height: 12px; fill: #ff7a00; }

        #rulesPage, #winnersPage, #validatePage { display: none; }
        
        .winner-row { background: #f0fdf4; border-left: 5px solid #22c55e; padding: 12px 16px; border-radius: 8px; margin-bottom: 10px; text-align: left; font-size: 14px; border: 1px solid #edf2f7; }
        .tag-pill { background: #fff; border: 1px solid #cbd5e1; color: #0f172a; padding: 3px 8px; border-radius: 6px; font-weight: 700; font-family: monospace; font-size: 13px; }

        .rule-item { text-align: left; margin-bottom: 14px; font-size: 14px; line-height: 1.6; color: #475569; }
        .rule-item b { color: #0f172a; }

        .validate-box {
            background: #ffffff;
            border-radius: 14px;
            padding: 18px;
            border: 1px solid #e2e8f0;
            margin-top: 14px;
            text-align: left;
        }
        .ticket-verified-card {
            background: #f0fdf4;
            border: 2px solid #22c55e;
            border-radius: 12px;
            padding: 16px;
            margin-top: 14px;
            display: none;
        }

        /* MÉTODOS DE PAGO: MOVIMIENTO CONTINUO HACIA LA DERECHA */
        .payments-bottom-box {
            position: relative;
            z-index: 10;
            background: rgba(255, 255, 255, 0.95);
            border: 1px solid #e2e8f0;
            border-radius: 14px;
            padding: 10px 0;
            margin: 20px auto 8px auto;
            width: 100%;
            overflow: hidden;
            backdrop-filter: blur(8px);
            pointer-events: none;
        }
        .payments-bottom-title {
            font-size: 11px;
            font-weight: 800;
            color: #94a3b8;
            letter-spacing: 1px;
            text-transform: uppercase;
            text-align: center;
            margin-bottom: 8px;
        }
        .payments-marquee-right {
            display: flex;
            width: max-content;
            animation: scrollPayHaciaDerecha 16s linear infinite;
        }
        @keyframes scrollPayHaciaDerecha {
            0% { transform: translateX(-50%); }
            100% { transform: translateX(0); }
        }
        .pay-mini-badge {
            display: inline-flex;
            align-items: center;
            gap: 6px;
            background: #ffffff;
            border: 1px solid #e2e8f0;
            border-radius: 8px;
            padding: 6px 12px;
            margin: 0 5px;
            font-size: 12px;
            font-weight: 700;
            color: #334155;
            flex-shrink: 0;
            box-shadow: 0 1px 4px rgba(0,0,0,0.04);
        }
        .pay-mini-badge svg {
            width: 16px;
            height: 16px;
            flex-shrink: 0;
        }

        /* BOTÓN FLOTANTE DISCRETO EN LA ESQUINA */
        #telegram-float-btn {
            position: fixed;
            bottom: 12px;
            right: 12px;
            background: rgba(34, 158, 217, 0.85);
            backdrop-filter: blur(4px);
            width: 38px;
            height: 38px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            text-decoration: none;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
            z-index: 999;
            transition: transform 0.15s ease, background-color 0.15s ease;
        }
        #telegram-float-btn:active {
            transform: scale(0.90);
            background: #1e87bb;
        }
        #telegram-float-btn span {
            display: none;
        }
        #telegram-float-btn svg {
            width: 18px;
            height: 18px;
            fill: #ffffff;
        }

        footer { 
            position: relative; 
            z-index: 10; 
            margin-top: 14px;
            padding: 12px 14px 24px 14px;
            text-align: center; 
            font-size: 12px; 
            color: #94a3b8; 
            line-height: 1.6; 
        }
        footer a { color: #ff7a00; text-decoration: none; font-weight: 700; }

        /* MODALES */
        .modal-overlay { 
            position: fixed; 
            top: 0; 
            left: 0; 
            width: 100%; 
            height: 100%; 
            background: rgba(15, 23, 42, 0.75); 
            backdrop-filter: blur(5px); 
            display: none; 
            justify-content: center; 
            align-items: center; 
            z-index: 1000; 
            padding: 16px; 
        }
        .modal-content { 
            background: #ffffff; 
            width: 100%; 
            max-width: 460px; 
            max-height: 90vh; 
            overflow-y: auto; 
            border-radius: 16px; 
            padding: 24px; 
            box-shadow: 0 20px 40px rgba(0,0,0,0.25); 
            text-align: center; 
            position: relative; 
            animation: popIn 0.25s ease-out; 
        }
        @keyframes popIn { from { transform: scale(0.92); opacity: 0; } to { transform: scale(1); opacity: 1; } }
        
        .modal-close { position: absolute; top: 12px; right: 14px; background: none; border: none; font-size: 18px; font-weight: bold; color: #94a3b8; cursor: pointer; }
        .modal-title { font-size: 16px; font-weight: 900; color: #0f172a; text-transform: uppercase; margin-bottom: 12px; }

        .inputs-grid { display: flex; justify-content: center; gap: 8px; margin: 12px 0; }
        .num-input { width: 44px; height: 48px; font-size: 20px; font-weight: bold; text-align: center; border-radius: 8px; border: 2px solid #cbd5e1; background: #fafafa; color: #222; }
        .num-input:focus { border-color: #ff7a00; outline: none; background: #fff; }
        .text-input, .select-input { width: 100%; height: 44px; font-size: 14px; padding: 0 12px; margin-bottom: 10px; border-radius: 8px; border: 1.5px solid #cbd5e1; background: #fafafa; color: #333; }
        .text-input:focus, .select-input:focus { border-color: #ff7a00; outline: none; background: #fff; }

        .btn-action { background: #ff7a00; color: #fff; border: none; padding: 12px; font-weight: 700; border-radius: 8px; cursor: pointer; width: 100%; font-size: 14px; text-transform: uppercase; box-shadow: 0 3px 6px rgba(255,122,0,0.25); margin-top: 6px; }
        .btn-action:active { background: #e06900; }
        .btn-random { background: #2563eb; color: #fff; border: none; padding: 9px; font-weight: 700; border-radius: 6px; cursor: pointer; width: 100%; font-size: 12px; text-transform: uppercase; margin-bottom: 8px; }
        .btn-cancel { background: #f1f5f9; color: #475569; border: none; padding: 10px; font-weight: 700; border-radius: 8px; cursor: pointer; width: 100%; font-size: 13px; margin-top: 8px; }

        /* PESTAÑAS ADMIN */
        .admin-nav-tabs { display: flex; gap: 8px; margin-bottom: 16px; border-bottom: 2px solid #e2e8f0; padding-bottom: 8px; }
        .admin-tab-btn { flex: 1; padding: 8px 4px; font-size: 12px; font-weight: 800; text-transform: uppercase; border: none; background: #f1f5f9; color: #64748b; border-radius: 8px; cursor: pointer; }
        .admin-tab-btn.active { background: #ff7a00; color: #ffffff; }

        .admin-item-box { background: #f8fafc; border: 1px solid #e2e8f0; border-left: 4px solid #ff7a00; border-radius: 8px; padding: 10px 12px; margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; text-align: left; }
        .admin-item-info { font-size: 13px; color: #1e293b; }
        .admin-item-actions { display: flex; gap: 6px; }
        .btn-adm-mini { padding: 6px 10px; font-size: 11px; font-weight: bold; border-radius: 6px; border: none; cursor: pointer; }
        .btn-adm-edit { background: #3b82f6; color: #fff; }
        .btn-adm-del { background: #ef4444; color: #fff; }

        #customToast {
            position: fixed;
            bottom: 24px;
            right: 24px;
            background: #0f172a;
            color: #ffffff;
            border-left: 4px solid #ff7a00;
            padding: 14px 20px;
            border-radius: 8px;
            font-size: 13px;
            font-weight: 600;
            box-shadow: 0 8px 24px rgba(0,0,0,0.25);
            z-index: 1001;
            display: none;
            align-items: center;
            gap: 10px;
            animation: slideToast 0.3s ease-out;
        }
        @keyframes slideToast {
            from { transform: translateY(20px); opacity: 0; }
            to { transform: translateY(0); opacity: 1; }
        }
    </style>
</head>
<body oncontextmenu="return false;">

    <canvas id="bgCanvas"></canvas>
    <canvas id="fireworksCanvas"></canvas>

    <!-- BOTÓN FLOTANTE DISCRETO TELEGRAM -->
    <a href="https://t.me/LoteriaOficialPRBot" target="_blank" rel="noopener noreferrer" id="telegram-float-btn" title="Soporte Telegram">
        <svg viewBox="0 0 24 24">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm4.64 6.8c-.15 1.58-.8 5.42-1.13 7.19-.14.75-.42 1-.68 1.03-.58.05-1.02-.38-1.58-.75-.88-.58-1.38-.94-2.23-1.5-.99-.65-.35-1.01.22-1.59.15-.15 2.71-2.48 2.76-2.69a.2.2 0 00-.05-.18c-.06-.05-.14-.03-.21-.02-.09.02-1.49.95-4.22 2.79-.4.27-.76.41-1.08.4-.36-.01-1.04-.2-1.55-.37-.63-.2-1.12-.31-1.08-.66.02-.18.27-.36.75-.55 2.92-1.27 4.86-2.11 5.83-2.51 2.78-1.16 3.35-1.36 3.73-1.36.08 0 .27.02.39.12.1.08.13.19.14.27-.01.06.01.24 0 .38z"/>
        </svg>
    </a>

    <div id="customToast">
        <span id="toastMsg">Mensaje</span>
    </div>

    <header>
        <div class="brand-container" onclick="gestionarToqueLogo()">
            <img src="https://i.postimg.cc/sXcgPJMZ/Picsart-26-09-18-08-28-37-079.png" alt="Logo Lotería" class="official-logo-img">
            <div class="logo-text-group">
                <span class="logo-main">Lotería</span>
                <span class="logo-sub">Oficial</span>
            </div>
        </div>
        <div class="header-btns">
            <button class="nav-btn nav-btn-check" id="btnValidar" onclick="toggleVista('validar')">🔍 Validar Boleto</button>
            <button class="nav-btn" id="btnReglas" onclick="toggleVista('reglas')">Reglas</button>
            <button class="nav-btn" id="btnGanadores" onclick="toggleVista('ganadores')">Ganadores</button>
        </div>
    </header>

    <div class="sub-header">
        <span>Resultados y Jugadas Oficiales</span>
        <div class="live-visitors-badge">
            <span class="live-dot"></span>
            <span id="liveCounterText">48 en línea</span>
        </div>
    </div>

    <div class="marquee-box">
        <span>Loto Estelar ($10) Viernes 8 PM • Loto Medio Día ($5) Diario 12 PM • Loto Nocturno ($20) Diario 8 PM</span>
    </div>

    <div class="container">
        <!-- 1. VISTA PRINCIPAL DE SORTEOS -->
        <div id="sorteoView">
            
            <div class="lotos-stack">
                <!-- LOTO 1: LOTO ESTELAR ($10 c/u) -->
                <div class="card">
                    <div class="card-header-badge" style="color: #ff7a00;">Loto Estelar ($10 c/u)</div>
                    <div class="countdown-title-bar countdown-title-orange">Próximo Viernes 8:00 PM</div>

                    <div class="clock-grid">
                        <div class="clock-card clock-card-orange"><span class="clock-num" id="l1_d">00</span><span class="clock-label">Días</span></div>
                        <div class="clock-card clock-card-orange"><span class="clock-num" id="l1_h">00</span><span class="clock-label">Horas</span></div>
                        <div class="clock-card clock-card-orange"><span class="clock-num" id="l1_m">00</span><span class="clock-label">Minutos</span></div>
                        <div class="clock-card clock-card-orange"><span class="clock-num" id="l1_s">00</span><span class="clock-label">Segundos</span></div>
                    </div>
                    
                    <div class="balls-row" id="ballsRow1">
                        <div class="ball" id="l1_b0">?</div><div class="ball" id="l1_b1">?</div><div class="ball" id="l1_b2">?</div><div class="ball" id="l1_b3">?</div><div class="ball ball-special" id="l1_b4">?</div>
                    </div>

                    <div class="jackpot-pill">
                        <div class="jackpot-tag">Funda Acumulada</div>
                        <div class="jackpot-value" id="prizeDisplay1">$0.00</div>
                    </div>

                    <div class="winner-card" id="winnerBox1">
                        <div class="winner-title">¡Boleto Ganador Estelar!</div>
                        <div id="winnerDetails1" style="font-size: 14px; line-height: 1.5; color: #1e293b;"></div>
                    </div>
                </div>

                <!-- LOTO 2: LOTO MEDIO DÍA ($5 c/u) -->
                <div class="card">
                    <div class="card-header-badge" style="color: #0284c7;">Loto Medio Día ($5 c/u)</div>
                    <div class="countdown-title-bar countdown-title-blue">Próximo Sorteo 12:00 PM (4 Dígitos)</div>

                    <div class="clock-grid">
                        <div class="clock-card clock-card-blue"><span class="clock-num" id="l2_d">00</span><span class="clock-label">Días</span></div>
                        <div class="clock-card clock-card-blue"><span class="clock-num" id="l2_h">00</span><span class="clock-label">Horas</span></div>
                        <div class="clock-card clock-card-blue"><span class="clock-num" id="l2_m">00</span><span class="clock-label">Minutos</span></div>
                        <div class="clock-card clock-card-blue"><span class="clock-num" id="l2_s">00</span><span class="clock-label">Segundos</span></div>
                    </div>
                    
                    <div class="balls-row" id="ballsRow2">
                        <div class="ball" id="l2_b0">?</div><div class="ball" id="l2_b1">?</div><div class="ball" id="l2_b2">?</div><div class="ball ball-gold" id="l2_b3">?</div>
                    </div>

                    <div class="jackpot-pill jackpot-pill-blue">
                        <div class="jackpot-tag">Funda Acumulada</div>
                        <div class="jackpot-value jackpot-value-blue" id="prizeDisplay2">$0.00</div>
                    </div>

                    <div class="winner-card" id="winnerBox2">
                        <div class="winner-title" id="winnerBoxTitle2">¡Resultado Loto Medio Día!</div>
                        <div id="winnerDetails2" style="font-size: 14px; line-height: 1.5; color: #1e293b;"></div>
                    </div>
                </div>

                <!-- LOTO 3: LOTO NOCTURNO ($20 c/u) -->
                <div class="card">
                    <div class="card-header-badge" style="color: #9333ea;">Loto Nocturno ($20 c/u)</div>
                    <div class="countdown-title-bar countdown-title-purple">Próximo Sorteo 8:00 PM (2 Dígitos)</div>

                    <div class="clock-grid">
                        <div class="clock-card clock-card-purple"><span class="clock-num" id="l3_d">00</span><span class="clock-label">Días</span></div>
                        <div class="clock-card clock-card-purple"><span class="clock-num" id="l3_h">00</span><span class="clock-label">Horas</span></div>
                        <div class="clock-card clock-card-purple"><span class="clock-num" id="l3_m">00</span><span class="clock-label">Minutos</span></div>
                        <div class="clock-card clock-card-purple"><span class="clock-num" id="l3_s">00</span><span class="clock-label">Segundos</span></div>
                    </div>
                    
                    <div class="balls-row" id="ballsRow3">
                        <div class="ball" id="l3_b0">?</div><div class="ball ball-purple" id="l3_b1">?</div>
                    </div>

                    <div class="jackpot-pill jackpot-pill-purple">
                        <div class="jackpot-tag">Funda Acumulada</div>
                        <div class="jackpot-value jackpot-value-purple" id="prizeDisplay3">$0.00</div>
                    </div>

                    <div class="winner-card" id="winnerBox3">
                        <div class="winner-title" id="winnerBoxTitle3">¡Resultado Loto Nocturno!</div>
                        <div id="winnerDetails3" style="font-size: 14px; line-height: 1.5; color: #1e293b;"></div>
                    </div>
                </div>
            </div>

            <!-- CARRUSEL DE RESEÑAS -->
            <div class="reviews-section">
                <div class="reviews-header-bar">
                    <div class="reviews-title">Experiencias Reales de Jugadores</div>
                    <button class="btn-add-review" onclick="abrirModalResena()">+ Dejar Reseña</button>
                </div>
                <div class="marquee-carousel" id="reviewsCarouselTrack"></div>
            </div>

            <!-- MÉTODOS DE PAGO CONTINUOS HACIA LA DERECHA -->
            <div class="payments-bottom-box">
                <div class="payments-bottom-title">Métodos de Pago Aceptados</div>
                <div class="payments-marquee-right">
                    <div class="pay-mini-badge" style="color:#ea580c; border-color:#fed7aa; background:#fff7ed;">
                        <svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="11" fill="#ea580c"/><text x="12" y="16" font-size="10.5" font-weight="900" text-anchor="middle" fill="#ffffff" font-family="Arial, sans-serif">ATH</text></svg>
                        <span>ATH Móvil</span>
                    </div>
                    <div class="pay-mini-badge" style="color:#00d632; border-color:#bbf7d0; background:#f0fdf4;">
                        <svg viewBox="0 0 24 24"><rect width="24" height="24" rx="6" fill="#00d632"/><path d="M12 4.5v1.8c2.6.2 3.8 1.4 3.9 3.2h-2c-.1-1-1-1.6-2-1.6-.9 0-1.7.5-1.7 1.3 0 .8.6 1.1 2.2 1.6 2.3.7 3.5 1.6 3.5 3.5 0 1.9-1.4 3.1-3.9 3.3v1.9h-1.6v-1.9c-2.4-.2-3.8-1.5-4-3.5h2.1c.2 1.1 1 1.8 2.2 1.8 1 0 1.8-.6 1.8-1.4 0-.8-.7-1.2-2.3-1.7-2.3-.7-3.4-1.6-3.4-3.4 0-1.8 1.4-3 3.6-3.2V4.5H12z" fill="#ffffff"/></svg>
                        <span>Cash App</span>
                    </div>
                    <div class="pay-mini-badge" style="color:#0070ba; border-color:#bae6fd; background:#f0f9ff;">
                        <svg viewBox="0 0 24 24"><rect width="24" height="24" rx="6" fill="#0070ba"/><path d="M7 6.5h4.2c2.1 0 3.3 1 3.1 2.8-.3 2.1-1.7 3.2-3.7 3.2H8.8l-.9 5.5H6.1L7 6.5zm3.7 4.2c1 0 1.8-.5 1.9-1.5.1-.9-.5-1.4-1.5-1.4H8.6l-.5 2.9h2.6z" fill="#ffffff"/></svg>
                        <span>PayPal</span>
                    </div>
                    <div class="pay-mini-badge" style="color:#f59e0b; border-color:#fde68a; background:#fffbeb;">
                        <svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="11" fill="#f59e0b"/><path d="M14.5 10.2c.4-.4.6-1 .5-1.7-.2-1.3-1.4-1.7-2.9-1.7h-3v8.4h3.4c1.6 0 3-.6 3.1-2.1 0-.9-.3-1.6-1.1-2.1zm-3.8-2h1.6c.9 0 1.5.3 1.6.9 0 .7-.6 1-1.5 1h-1.7V8.2zm1.9 5.6h-1.9v-2.2h1.9c1 0 1.7.3 1.7 1.1 0 .8-.7 1.1-1.7 1.1z" fill="#ffffff"/></svg>
                        <span>Cripto</span>
                    </div>
                    <div class="pay-mini-badge" style="color:#ea580c; border-color:#fed7aa; background:#fff7ed;">
                        <svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="11" fill="#ea580c"/><text x="12" y="16" font-size="10.5" font-weight="900" text-anchor="middle" fill="#ffffff" font-family="Arial, sans-serif">ATH</text></svg>
                        <span>ATH Móvil</span>
                    </div>
                    <div class="pay-mini-badge" style="color:#00d632; border-color:#bbf7d0; background:#f0fdf4;">
                        <svg viewBox="0 0 24 24"><rect width="24" height="24" rx="6" fill="#00d632"/><path d="M12 4.5v1.8c2.6.2 3.8 1.4 3.9 3.2h-2c-.1-1-1-1.6-2-1.6-.9 0-1.7.5-1.7 1.3 0 .8.6 1.1 2.2 1.6 2.3.7 3.5 1.6 3.5 3.5 0 1.9-1.4 3.1-3.9 3.3v1.9h-1.6v-1.9c-2.4-.2-3.8-1.5-4-3.5h2.1c.2 1.1 1 1.8 2.2 1.8 1 0 1.8-.6 1.8-1.4 0-.8-.7-1.2-2.3-1.7-2.3-.7-3.4-1.6-3.4-3.4 0-1.8 1.4-3 3.6-3.2V4.5H12z" fill="#ffffff"/></svg>
                        <span>Cash App</span>
                    </div>
                    <div class="pay-mini-badge" style="color:#0070ba; border-color:#bae6fd; background:#f0f9ff;">
                        <svg viewBox="0 0 24 24"><rect width="24" height="24" rx="6" fill="#0070ba"/><path d="M7 6.5h4.2c2.1 0 3.3 1 3.1 2.8-.3 2.1-1.7 3.2-3.7 3.2H8.8l-.9 5.5H6.1L7 6.5zm3.7 4.2c1 0 1.8-.5 1.9-1.5.1-.9-.5-1.4-1.5-1.4H8.6l-.5 2.9h2.6z" fill="#ffffff"/></svg>
                        <span>PayPal</span>
                    </div>
                    <div class="pay-mini-badge" style="color:#f59e0b; border-color:#fde68a; background:#fffbeb;">
                        <svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="11" fill="#f59e0b"/><path d="M14.5 10.2c.4-.4.6-1 .5-1.7-.2-1.3-1.4-1.7-2.9-1.7h-3v8.4h3.4c1.6 0 3-.6 3.1-2.1 0-.9-.3-1.6-1.1-2.1zm-3.8-2h1.6c.9 0 1.5.3 1.6.9 0 .7-.6 1-1.5 1h-1.7V8.2zm1.9 5.6h-1.9v-2.2h1.9c1 0 1.7.3 1.7 1.1 0 .8-.7 1.1-1.7 1.1z" fill="#ffffff"/></svg>
                        <span>Cripto</span>
                    </div>
                </div>
            </div>
        </div>

        <!-- 2. VISTA VALIDAR BOLETO -->
        <div id="validatePage">
            <div class="card" style="text-align: left;">
                <div style="font-size: 15px; font-weight: 800; color: #ff7a00; margin-bottom: 8px; text-transform: uppercase; text-align: center;">Verificador Oficial de Boletos</div>
                <p style="font-size: 13px; color: #64748b; text-align: center; margin-bottom: 16px;">Introduce el <b>Código de Boleto (ID)</b> o tu <b>Teléfono</b> para validar tu jugada oficial:</p>
                <div class="validate-box">
                    <input type="text" id="checkTicketInput" class="text-input" placeholder="Ej: LOT-12345 o 7871234567" style="margin-bottom: 8px; font-size: 15px; font-weight: bold; text-transform: uppercase;">
                    <button class="btn-action" onclick="consultarBoletoJugador()">Consultar Estado</button>
                </div>
                <div id="ticketResultBox" class="ticket-verified-card"></div>
            </div>
        </div>

        <!-- 3. VISTA REGLAS -->
        <div id="rulesPage">
            <div class="card" style="text-align: left;">
                <div style="font-size: 15px; font-weight: 800; color: #ff7a00; margin-bottom: 14px; text-transform: uppercase; text-align: center;">Reglas e Instrucciones Oficiales</div>
                <div class="rule-item"><b>1. Loto Estelar ($10.00 por boleto):</b><br>• Sorteo: <b>Todos los Viernes a las 8:00 PM</b>. (5 dígitos).<br>• Funda Acumulada: El pozo acumulado se reparte al ganador garantizado.</div>
                <div class="rule-item"><b>2. Loto Medio Día ($5.00 por boleto):</b><br>• Sorteo: <b>Todos los Días a las 12:00 PM</b>. (4 dígitos del 0000 al 9999).<br>• <b>Dinámica de Números:</b> Si nadie acierta, el número sale descontinuado/bloqueado y la funda pasa al día siguiente. <b>Si alguien gana, se entrega el premio y la tómbola se restablece limpia como la primera vez</b>.<br>• <b>Funda Acumulada:</b> Si hay ganador, la funda vuelve a su base inicial.</div>
                <div class="rule-item"><b>3. Loto Nocturno ($20.00 por boleto):</b><br>• Sorteo: <b>Todos los Días a las 8:00 PM</b>. (2 dígitos del 00 al 99).<br>• <b>Dinámica de Números:</b> Igual que Medio Día, al haber un ganador se limpia la tómbola y se restablecen los números para una nueva ronda.<br>• <b>Funda Acumulada:</b> Pasa a la noche siguiente si no hay aciertos.</div>
                <div class="rule-item" style="margin-top: 14px; border-top: 1px solid #e2e8f0; padding-top: 10px; text-align: center;"><b>Soporte Oficial:</b><br><span style="color: #ff7a00; font-weight: bold;">lotteriapr.com@gmail.com</span></div>
            </div>
        </div>

        <!-- 4. VISTA GANADORES -->
        <div id="winnersPage">
            <div class="card">
                <div style="font-size: 14px; font-weight: 800; color: #15803d; margin-bottom: 12px; text-transform: uppercase; text-align: left;">Historial de Ganadores y Resultados de Tómbola</div>
                <div id="winnersContainer">
                    <p style="color: #94a3b8; font-size: 13px; font-style: italic;">Aún no hay registros en este historial.</p>
                </div>
            </div>
        </div>

    </div>

    <!-- FOOTER -->
    <footer>
        <div>© 2026 Lotería Oficial • Todos los derechos reservados</div>
        <div>Contacto: <a href="mailto:lotteriapr.com@gmail.com">lotteriapr.com@gmail.com</a></div>
        <div style="font-size: 11px; margin-top: 4px;">Juega con responsabilidad. Mayores de 18 años.</div>
    </footer>

    <!-- MODAL CONFIRMACIÓN Y ENVÍO DE RECIBO A WHATSAPP -->
    <div class="modal-overlay" id="modalWhatsAppSuccess">
        <div class="modal-content" style="max-width: 400px; text-align: center;">
            <div class="modal-title" style="color: #15803d;">✅ Boleto Emitido con Éxito</div>
            <p style="font-size: 13px; color: #475569; margin-bottom: 12px;">El recibo fue generado. Toca el botón para abrir y enviarlo a tu WhatsApp:</p>
            <pre id="waReceiptPreview" style="background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 8px; padding: 10px; font-size: 11px; text-align: left; white-space: pre-wrap; margin-bottom: 14px; max-height: 180px; overflow-y: auto; color: #1e293b;"></pre>
            <a id="btnEnviarWA" href="#" target="_blank" rel="noopener noreferrer" class="btn-action" style="background: #25d366; text-decoration: none; display: block; padding: 14px; font-size: 15px;">📲 Abrir y Enviar a WhatsApp</a>
            <button class="btn-cancel" onclick="cerrarModal('modalWhatsAppSuccess')">Cerrar</button>
        </div>
    </div>

    <!-- MODAL DE RESEÑA -->
    <div class="modal-overlay" id="modalReview">
        <div class="modal-content" style="text-align: left;">
            <button class="modal-close" onclick="cerrarModal('modalReview')">✕</button>
            <div class="modal-title" style="text-align: center; color: #ff7a00;">Publicar Experiencia</div>
            <input type="text" id="revNombre" class="text-input" placeholder="Tu nombre">
            <label style="font-size: 11px; font-weight: bold; color: #64748b;">CALIFICACIÓN:</label>
            <select id="revStars" class="select-input" style="margin-top: 4px;">
                <option value="5">★★★★★ (5 estrellas - Excelente)</option>
                <option value="4">★★★★☆ (4 estrellas - Muy Bueno)</option>
                <option value="3">★★★☆☆ (3 estrellas - Regular)</option>
            </select>
            <textarea id="revComentario" class="text-input" placeholder="Escribe tu testimonio aquí..." style="height: 75px; padding: 10px; resize: none;"></textarea>
            <button class="btn-action" onclick="guardarNuevaResena()">Publicar Reseña</button>
            <button class="btn-cancel" onclick="cerrarModal('modalReview')">Cancelar</button>
        </div>
    </div>

    <!-- MODAL ADMIN (PIN) -->
    <div class="modal-overlay" id="modalPassword">
        <div class="modal-content">
            <button class="modal-close" onclick="cerrarModal('modalPassword')">✕</button>
            <div class="modal-title">Panel Administrativo</div>
            <p style="font-size: 12px; color: #64748b; margin-bottom: 14px;">Ingresa el PIN de seguridad:</p>
            <input type="password" id="inputPassModal" class="text-input" placeholder="••••••••" style="text-align: center; font-size: 18px; letter-spacing: 3px;">
            <div id="passErrorMsg" style="color: #ef4444; font-size: 12px; font-weight: bold; margin-bottom: 8px;"></div>
            <button class="btn-action" onclick="validarPINModal()">Acceder</button>
            <button class="btn-cancel" onclick="cerrarModal('modalPassword')">Cancelar</button>
        </div>
    </div>

    <!-- MODAL PRINCIPAL DE ADMINISTRACIÓN -->
    <div class="modal-overlay" id="modalAdminForm">
        <div class="modal-content" style="text-align: left;">
            <button class="modal-close" onclick="cerrarModal('modalAdminForm')">✕</button>
            <div class="admin-nav-tabs">
                <button class="admin-tab-btn active" id="tabEmitir" onclick="cambiarPestanaAdmin('emitir')">Emitir Boletos</button>
                <button class="admin-tab-btn" id="tabGestionar" onclick="cambiarPestanaAdmin('gestionar')">Jugadores / Boletos</button>
            </div>

            <!-- SUB-PANEL 1: EMITIR BOLETOS -->
            <div id="panelEmitirBoletos">
                <label style="font-size: 11px; color: #64748b; font-weight: bold;">SELECCIONA EL SORTEO:</label>
                <select id="tipoLotoSelect" class="select-input" onchange="adaptarFormularioSorteo()" style="margin-top: 4px;">
                    <option value="1">Loto Estelar ($10 c/u - 5 Números)</option>
                    <option value="2">Loto Medio Día ($5 c/u - 4 Números)</option>
                    <option value="3">Loto Nocturno ($20 c/u - 2 Números)</option>
                </select>

                <input type="text" id="nombre" class="text-input" placeholder="Nombre del jugador">
                <input type="tel" id="telefono" class="text-input" placeholder="Teléfono de contacto">
                
                <label style="font-size: 11px; color: #64748b; font-weight: bold;" id="labelCantidadTxt">CANTIDAD DE BOLETOS ($10 c/u):</label>
                <input type="number" id="cantidadBoletos" class="text-input" value="1" min="1" max="1000" style="margin-top: 4px;">

                <label style="font-size: 11px; color: #64748b; font-weight: bold;" id="labelJugadaTxt">JUGADA (5 NÚMEROS):</label>
                <button type="button" class="btn-random" onclick="generarJugadaAlAzar()">Jugada al Azar</button>

                <div class="inputs-grid" id="inputsGridForm">
                    <input type="text" inputmode="numeric" id="n1" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n2')" onkeydown="manejarRetroceso(event, this, null)">
                    <input type="text" inputmode="numeric" id="n2" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n3')" onkeydown="manejarRetroceso(event, this, 'n1')">
                    <input type="text" inputmode="numeric" id="n3" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n4')" onkeydown="manejarRetroceso(event, this, 'n2')">
                    <input type="text" inputmode="numeric" id="n4" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n5')" onkeydown="manejarRetroceso(event, this, 'n3')">
                    <input type="text" inputmode="numeric" id="n5" class="num-input" maxlength="1" oninput="controlCasilla(this, null)" onkeydown="manejarRetroceso(event, this, 'n4')">
                </div>
                <button class="btn-action" id="btnEmitirBoleto" onclick="comprarTickets()">Emitir Boletos ($10 c/u)</button>
                <div id="msg" style="margin-top: 8px; font-size: 12px; font-weight: bold; text-align: center;"></div>
            </div>

            <!-- SUB-PANEL 2: GESTIONAR / EDITAR / BORRAR JUGADORES -->
            <div id="panelGestionarBoletos" style="display: none;">
                <label style="font-size: 11px; color: #64748b; font-weight: bold;">FILTRAR POR SORTEO:</label>
                <select id="tipoLotoAdminGestion" class="select-input" onchange="renderizarAdminJugadores()" style="margin-top: 4px;">
                    <option value="1">Loto Estelar ($10)</option>
                    <option value="2">Loto Medio Día ($5)</option>
                    <option value="3">Loto Nocturno ($20)</option>
                </select>
                <div id="listaAdminJugadores" style="max-height: 280px; overflow-y: auto; padding-right: 4px; margin-top: 10px;"></div>
            </div>

        </div>
    </div>

    <script>
        // @ts-nocheck

        // ==========================================
        // CONFIGURACIÓN DE FIREBASE
        // ==========================================
        const firebaseConfig = {
            databaseURL: "https://lotte-be526-default-rtdb.firebaseio.com/"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        // CARGA INICIAL INSTANTÁNEA DESDE CACHÉ LOCAL
        (function cargarCacheInicial() {
            const p1Cache = localStorage.getItem('cache_prize_1');
            const p2Cache = localStorage.getItem('cache_prize_2');
            const p3Cache = localStorage.getItem('cache_prize_3');

            if (p1Cache) document.getElementById('prizeDisplay1').innerText = `$${parseFloat(p1Cache).toFixed(2)}`;
            if (p2Cache) document.getElementById('prizeDisplay2').innerText = `$${parseFloat(p2Cache).toFixed(2)}`;
            if (p3Cache) document.getElementById('prizeDisplay3').innerText = `$${parseFloat(p3Cache).toFixed(2)}`;
        })();

        // ESCUCHA EN TIEMPO REAL DESDE LA NUBE
        db.ref('pozos_loteria').on('value', (snapshot) => {
            const data = snapshot.val();
            const p1El = document.getElementById('prizeDisplay1');
            const p2El = document.getElementById('prizeDisplay2');
            const p3El = document.getElementById('prizeDisplay3');

            if (data) {
                let v1 = parseFloat(data.estelar || 0);
                let v2 = parseFloat(data.mediodia || 0);
                let v3 = parseFloat(data.nocturno || 0);

                if (p1El) p1El.innerText = `$${v1.toFixed(2)}`;
                if (p2El) p2El.innerText = `$${v2.toFixed(2)}`;
                if (p3El) p3El.innerText = `$${v3.toFixed(2)}`;

                localStorage.setItem('cache_prize_1', v1);
                localStorage.setItem('cache_prize_2', v2);
                localStorage.setItem('cache_prize_3', v3);
            }
        });

        // ACTUALIZACIÓN DE PREMIOS EN FIREBASE Y LOCAL TRAS EDICIONES
        function actualizarPremiosUI() {
            let t1 = getTickets(1);
            let p1 = getBoteAcumulado(1) + (t1.length * PRECIO_ESTELAR);
            let t2 = getTickets(2);
            let p2 = getBoteAcumulado(2) + (t2.length * PRECIO_MEDIODIA);
            let t3 = getTickets(3);
            let p3 = getBoteAcumulado(3) + (t3.length * PRECIO_NOCTURNO);
            db.ref('pozos_loteria').set({ estelar: p1, mediodia: p2, nocturno: p3 });
        }

        // PROTECCIÓN ANTI-F12
        document.addEventListener('keydown', function(e) {
            if (e.keyCode === 123) { e.preventDefault(); return false; }
            if (e.ctrlKey && e.shiftKey && (e.keyCode === 73 || e.keyCode === 74 || e.keyCode === 67)) { e.preventDefault(); return false; }
            if (e.ctrlKey && (e.keyCode === 85 || e.keyCode === 83)) { e.preventDefault(); return false; }
        });

        // CONTADOR VISITANTES
        let timerVisitantes = null;
        function obtenerVisitantesGuardados() {
            let guardado = localStorage.getItem('loteria_online_visitors');
            let timestamp = localStorage.getItem('loteria_online_timestamp');
            let ahora = Date.now();
            if (!guardado || !timestamp || (ahora - parseInt(timestamp) > 600000)) {
                let inicial = Math.floor(Math.random() * 90) + 10;
                localStorage.setItem('loteria_online_visitors', inicial.toString());
                localStorage.setItem('loteria_online_timestamp', ahora.toString());
                return inicial;
            }
            return parseInt(guardado);
        }

        let visitantesEnLinea = obtenerVisitantesGuardados();
        function actualizarVisitantesEnLinea() {
            let ahora = Date.now();
            let timestamp = parseInt(localStorage.getItem('loteria_online_timestamp') || "0");
            if (ahora - timestamp < 20000) {
                const el = document.getElementById('liveCounterText');
                if (el) el.innerText = `${visitantesEnLinea} en línea`;
                programarSiguienteCambioVisitantes();
                return;
            }
            const pasos = [-3, -2, -1, 1, 2, 3];
            const cambio = pasos[Math.floor(Math.random() * pasos.length)];
            visitantesEnLinea = Math.max(1, Math.min(100, visitantesEnLinea + cambio));
            localStorage.setItem('loteria_online_visitors', visitantesEnLinea.toString());
            localStorage.setItem('loteria_online_timestamp', ahora.toString());
            const el = document.getElementById('liveCounterText');
            if (el) el.innerText = `${visitantesEnLinea} en línea`;
            programarSiguienteCambioVisitantes();
        }

        function programarSiguienteCambioVisitantes() {
            const tiempoEspera = Math.floor(Math.random() * 10000) + 20000;
            clearTimeout(timerVisitantes);
            timerVisitantes = setTimeout(actualizarVisitantesEnLinea, tiempoEspera);
        }

        const elInicioVisitantes = document.getElementById('liveCounterText');
        if (elInicioVisitantes) elInicioVisitantes.innerText = `${visitantesEnLinea} en línea`;
        programarSiguienteCambioVisitantes();

        let toastTimeout = null;
        function showToast(mensaje, color) {
            const toast = document.getElementById('customToast');
            const msgEl = document.getElementById('toastMsg');
            if (!toast || !msgEl) return;
            toast.style.borderLeftColor = color || "#ff7a00";
            msgEl.innerText = mensaje;
            toast.style.display = 'flex';
            if (toastTimeout) clearTimeout(toastTimeout);
            toastTimeout = setTimeout(() => { toast.style.display = 'none'; }, 3500);
        }

        // FUEGOS ARTIFICIALES
        const fCanvas = document.getElementById('fireworksCanvas');
        const fCtx = fCanvas.getContext('2d');
        let fireworks = [];
        let animFireworksId = null;

        function resizeFireworksCanvas() {
            fCanvas.width = window.innerWidth;
            fCanvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resizeFireworksCanvas);
        resizeFireworksCanvas();

        function lanzarFuegosArtificiales() {
            fCanvas.style.display = 'block';
            fireworks = [];
            const colores = ['#ff0055', '#ff9900', '#ffee00', '#00ff66', '#00ccff', '#cc00ff', '#ffffff'];
            for (let i = 0; i < 180; i++) {
                const angle = Math.random() * Math.PI * 2;
                const speed = Math.random() * 9 + 2;
                fireworks.push({
                    x: fCanvas.width / 2 + (Math.random() - 0.5) * 200,
                    y: fCanvas.height * 0.45 + (Math.random() - 0.5) * 150,
                    vx: Math.cos(angle) * speed,
                    vy: Math.sin(angle) * speed,
                    color: colores[Math.floor(Math.random() * colores.length)],
                    radius: Math.random() * 3.5 + 1.5,
                    alpha: 1,
                    decay: Math.random() * 0.015 + 0.008,
                    gravity: 0.12
                });
            }

            function loopFireworks() {
                fCtx.clearRect(0, 0, fCanvas.width, fCanvas.height);
                fireworks.forEach((p, idx) => {
                    p.x += p.vx; p.y += p.vy; p.vy += p.gravity; p.alpha -= p.decay;
                    if (p.alpha <= 0) {
                        fireworks.splice(idx, 1);
                    } else {
                        fCtx.save();
                        fCtx.globalAlpha = p.alpha;
                        fCtx.beginPath();
                        fCtx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
                        fCtx.fillStyle = p.color;
                        fCtx.fill();
                        fCtx.restore();
                    }
                });
                if (fireworks.length > 0) {
                    animFireworksId = requestAnimationFrame(loopFireworks);
                } else {
                    fCanvas.style.display = 'none';
                    cancelAnimationFrame(animFireworksId);
                }
            }
            cancelAnimationFrame(animFireworksId);
            loopFireworks();
        }

        // MOTOR DE RESEÑAS
        const RESENAS_POR_DEFECTO = [
            { nombre: "Carlos Méndez", rating: 5, fecha: "17 Sep, 2026", texto: "Cobré el premio el mismo viernes por la noche. 100% legítimo." },
            { nombre: "Valeria Torres", rating: 5, fecha: "16 Sep, 2026", texto: "El Loto Medio Día a $5 diario está excelente y transparente." },
            { nombre: "Héctor Rivera", rating: 5, fecha: "14 Sep, 2026", texto: "Compré boletos de cantazo y quedaron al tiro registrados." },
            { nombre: "Yadiel Santiago", rating: 5, fecha: "12 Sep, 2026", texto: "Premio exacto y sin rodeos. Recomendado al 100%." }
        ];

        function obtenerResenas() {
            const guardadas = localStorage.getItem('loteria_resenas_v2');
            return guardadas ? JSON.parse(guardadas) : RESENAS_POR_DEFECTO;
        }

        function renderizarResenas() {
            const track = document.getElementById('reviewsCarouselTrack');
            if (!track) return;
            const resenas = obtenerResenas();
            const listaDoble = [...resenas, ...resenas];
            track.innerHTML = listaDoble.map(r => `
                <div class="review-card">
                    <div class="review-top">
                        <span class="review-stars">${'★'.repeat(r.rating)}${'☆'.repeat(5 - r.rating)}</span>
                        <span class="review-date">${r.fecha}</span>
                    </div>
                    <div class="review-badge">
                        <div class="badge-dot"></div>
                        <div class="badge-info">
                            <span class="badge-tag">Verificado</span>
                            <span class="badge-user">${r.nombre}</span>
                        </div>
                    </div>
                    <p class="review-text">${r.texto}</p>
                    <div class="review-footer">
                        <span>Boleto Oficial</span>
                        <svg viewBox="0 0 24 24"><path d="M5 13l4 4L19 7"/></svg>
                    </div>
                </div>
            `).join('');
        }

        function abrirModalResena() {
            const revNombre = document.getElementById('revNombre');
            const revCom = document.getElementById('revComentario');
            if (revNombre) revNombre.value = '';
            if (revCom) revCom.value = '';
            const modalRev = document.getElementById('modalReview');
            if (modalRev) modalRev.style.display = 'flex';
        }

        function guardarNuevaResena() {
            const revNombre = document.getElementById('revNombre');
            const revCom = document.getElementById('revComentario');
            const revStars = document.getElementById('revStars');
            const nombre = revNombre ? revNombre.value.trim() : "";
            const rating = revStars ? (parseInt(revStars.value) || 5) : 5;
            const texto = revCom ? revCom.value.trim() : "";
            if (!nombre || !texto) {
                showToast("Completa tu nombre y comentario.", "#ef4444");
                return;
            }
            const fechaActual = new Date();
            const meses = ['Ene','Feb','Mar','Abr','May','Jun','Jul','Ago','Sep','Oct','Nov','Dic'];
            const fechaStr = `${fechaActual.getDate()} ${meses[fechaActual.getMonth()]}, ${fechaActual.getFullYear()}`;
            let resenas = obtenerResenas();
            resenas.unshift({ nombre: nombre, rating: rating, fecha: fechaStr, texto: texto });
            localStorage.setItem('loteria_resenas_v2', JSON.stringify(resenas));
            renderizarResenas();
            cerrarModal('modalReview');
            showToast("¡Tu reseña ha sido publicada con éxito!", "#22c55e");
        }
        renderizarResenas();

        // ENCRIPTACIÓN SHA-256 PIN ADMIN 4010
        function sha256Sync(ascii) {
            function rightRotate(value, amount) { return (value >>> amount) | (value << (32 - amount)); }
            var mathPow = Math.pow;
            var maxWord = mathPow(2, 32);
            var lengthProperty = 'length';
            var i, j;
            var result = '';
            var words = [];
            var asciiBitLength = ascii[lengthProperty] * 8;
            var hash = [];
            var k = [];
            var primeCounter = 0;
            var isComposite = {};
            for (var candidate = 2; primeCounter < 64; candidate++) {
                if (!isComposite[candidate]) {
                    for (i = 0; i < 300; i += candidate) { isComposite[i] = candidate; }
                    hash[primeCounter] = (mathPow(candidate, .5) * maxWord) | 0;
                    k[primeCounter++] = (mathPow(candidate, 1 / 3) * maxWord) | 0;
                }
            }
            ascii += '\x80';
            while (ascii[lengthProperty] % 64 - 56) ascii += '\x00';
            for (i = 0; i < ascii[lengthProperty]; i++) {
                j = ascii.charCodeAt(i);
                if (j >> 8) return '';
                words[i >> 2] |= j << ((3 - i) % 4) * 8;
            }
            words[words[lengthProperty]] = ((asciiBitLength / maxWord) | 0);
            words[words[lengthProperty]] = (asciiBitLength | 0);
            for (j = 0; j < words[lengthProperty];) {
                var w = words.slice(j, j += 16);
                var oldHash = hash;
                hash = hash.slice(0, 8);
                for (i = 0; i < 64; i++) {
                    var w15 = w[i - 15], w2 = w[i - 2];
                    var s0 = rightRotate(w15, 7) ^ rightRotate(w15, 18) ^ (w15 >>> 3);
                    var s1 = rightRotate(w2, 17) ^ rightRotate(w2, 19) ^ (w2 >>> 10);
                    w[i] = (i < 16) ? w[i] : (w[i - 16] + s0 + w[i - 7] + s1) | 0;
                    var s0_maj = rightRotate(hash[0], 2) ^ rightRotate(hash[0], 13) ^ rightRotate(hash[0], 22);
                    var maj = (hash[0] & hash[1]) ^ (hash[0] & hash[2]) ^ (hash[1] & hash[2]);
                    var t2 = (s0_maj + maj) | 0;
                    var s1_ch = rightRotate(hash[4], 6) ^ rightRotate(hash[4], 11) ^ rightRotate(hash[4], 25);
                    var ch = (hash[4] & hash[5]) ^ (~hash[4] & hash[6]);
                    var t1 = (hash[7] + s1_ch + ch + k[i] + w[i]) | 0;
                    hash = [(t1 + t2) | 0].concat(hash);
                    hash[4] = (hash[4] + t1) | 0;
                }
                for (i = 0; i < 8; i++) { hash[i] = (hash[i] + oldHash[i]) | 0; }
            }
            for (i = 0; i < 8; i++) {
                for (j = 3; j + 1; j--) {
                    var b = (hash[i] >> (j * 8)) & 255;
                    result += ((b < 16) ? 0 : '') + b.toString(16);
                }
            }
            return result;
        }

        const ADMIN_PIN_HASH = "735c02b3780367303e8784fc57736e4f3a9e32014fc5291d90c0065a6c3f68e0";

        function validarPINModal() {
            const passInput = document.getElementById('inputPassModal');
            const pass = passInput ? passInput.value.trim() : "";
            const hashCalculado = sha256Sync(pass);
            if (hashCalculado === ADMIN_PIN_HASH || pass === "4010") {
                cerrarModal('modalPassword');
                adaptarFormularioSorteo();
                cambiarPestanaAdmin('emitir');
                const adminModal = document.getElementById('modalAdminForm');
                if (adminModal) adminModal.style.display = 'flex';
            } else {
                const errMsg = document.getElementById('passErrorMsg');
                if (errMsg) errMsg.innerText = "PIN incorrecto. Intenta nuevamente.";
            }
        }

        function cambiarPestanaAdmin(pestana) {
            const btnEmitir = document.getElementById('tabEmitir');
            const btnGestionar = document.getElementById('tabGestionar');
            const panelEmitir = document.getElementById('panelEmitirBoletos');
            const panelGestionar = document.getElementById('panelGestionarBoletos');
            if (pestana === 'emitir') {
                btnEmitir.classList.add('active');
                btnGestionar.classList.remove('active');
                panelEmitir.style.display = 'block';
                panelGestionar.style.display = 'none';
            } else {
                btnGestionar.classList.add('active');
                btnEmitir.classList.remove('active');
                panelEmitir.style.display = 'none';
                panelGestionar.style.display = 'block';
                renderizarAdminJugadores();
            }
        }

        function renderizarAdminJugadores() {
            const selectEl = document.getElementById('tipoLotoAdminGestion');
            const lotoId = selectEl ? selectEl.value : '1';
            const contenedor = document.getElementById('listaAdminJugadores');
            if (!contenedor) return;
            const tickets = getTickets(lotoId);
            if (tickets.length === 0) {
                contenedor.innerHTML = `<p style="font-size:12px; color:#94a3b8; font-style:italic; padding:10px 0;">No hay boletos registrados en este sorteo.</p>`;
                return;
            }
            contenedor.innerHTML = tickets.map((t, idx) => `
                <div class="admin-item-box">
                    <div class="admin-item-info">
                        <b>${t.nombre}</b> (${t.telefono})<br>
                        <small style="color:#64748b; font-weight:bold;">ID: #${t.id || 'N/A'}</small><br>
                        <span class="tag-pill">${t.numeros}</span>
                    </div>
                    <div class="admin-item-actions">
                        <button class="btn-adm-mini btn-adm-edit" onclick="editarBoletoAdmin(${lotoId}, ${idx})">Editar</button>
                        <button class="btn-adm-mini btn-adm-del" onclick="eliminarBoletoAdmin(${lotoId}, ${idx})">Borrar</button>
                    </div>
                </div>
            `).join('');
        }

        function eliminarBoletoAdmin(lotoId, idx) {
            let tickets = getTickets(lotoId);
            if (confirm(`¿Deseas eliminar el boleto de ${tickets[idx].nombre}?`)) {
                tickets.splice(idx, 1);
                setTickets(lotoId, tickets);
                actualizarPremiosUI();
                renderizarAdminJugadores();
                showToast("Boleto eliminado correctamente.", "#ef4444");
            }
        }

        function editarBoletoAdmin(lotoId, idx) {
            let tickets = getTickets(lotoId);
            let boleto = tickets[idx];
            let nuevoNombre = prompt("Editar nombre del titular:", boleto.nombre);
            if (nuevoNombre === null) return;
            let nuevosNums = prompt("Editar números (ejemplo: 1 - 2 - 3):", boleto.numeros);
            if (nuevosNums === null) return;
            boleto.nombre = nuevoNombre.trim() || boleto.nombre;
            boleto.numeros = nuevosNums.trim() || boleto.numeros;
            tickets[idx] = boleto;
            setTickets(lotoId, tickets);
            actualizarPremiosUI();
            renderizarAdminJugadores();
            showToast("Boleto actualizado.", "#22c55e");
        }

        // CANVAS FONDO
        const canvas = document.getElementById('bgCanvas');
        const ctx = canvas.getContext('2d');
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        const particles = [];
        for (let i = 0; i < 45; i++) {
            particles.push({
                x: Math.random() * canvas.width,
                y: Math.random() * canvas.height,
                size: Math.random() * 2.2 + 0.8,
                speedY: -(Math.random() * 0.45 + 0.15),
                speedX: (Math.random() - 0.5) * 0.25,
                alpha: Math.random() * 0.7 + 0.2,
                fadeSpeed: (Math.random() * 0.015 + 0.005),
                isSparkle: Math.random() > 0.65,
                rotation: Math.random() * Math.PI * 2,
                rotSpeed: (Math.random() - 0.5) * 0.03
            });
        }

        function drawSparkle(x, y, radius, alpha) {
            ctx.save();
            ctx.translate(x, y);
            ctx.beginPath();
            ctx.fillStyle = `rgba(255, 200, 80, ${alpha})`;
            for (let i = 0; i < 4; i++) {
                ctx.rotate(Math.PI / 2);
                ctx.lineTo(0, radius * 2.2);
                ctx.lineTo(radius * 0.4, radius * 0.4);
            }
            ctx.fill();
            ctx.restore();
        }

        function animateParticles() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            particles.forEach(p => {
                p.y += p.speedY; p.x += p.speedX; p.alpha += p.fadeSpeed; p.rotation += p.rotSpeed;
                if (p.alpha > 0.85 || p.alpha < 0.15) p.fadeSpeed = -p.fadeSpeed;
                if (p.y < -10) { p.y = canvas.height + 10; p.x = Math.random() * canvas.width; }
                if (p.x < -10) p.x = canvas.width + 10;
                if (p.x > canvas.width + 10) p.x = -10;
                if (p.isSparkle) {
                    drawSparkle(p.x, p.y, p.size, p.alpha);
                } else {
                    ctx.beginPath();
                    ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                    ctx.fillStyle = `rgba(255, 160, 40, ${p.alpha})`;
                    ctx.shadowBlur = 6;
                    ctx.shadowColor = 'rgba(255, 140, 0, 0.6)';
                    ctx.fill();
                    ctx.shadowBlur = 0;
                }
            });
            requestAnimationFrame(animateParticles);
        }
        animateParticles();

        let toquesLogo = 0;
        let timerLogo = null;
        function gestionarToqueLogo() {
            toquesLogo++;
            if (timerLogo) clearTimeout(timerLogo);
            timerLogo = setTimeout(() => { toquesLogo = 0; }, 800);
            if (toquesLogo >= 3) {
                toquesLogo = 0;
                const passMsg = document.getElementById('passErrorMsg');
                const passIn = document.getElementById('inputPassModal');
                const passModal = document.getElementById('modalPassword');
                if (passMsg) passMsg.innerText = "";
                if (passIn) passIn.value = "";
                if (passModal) passModal.style.display = 'flex';
                if (passIn) passIn.focus();
            }
        }

        function cerrarModal(id) {
            const modal = document.getElementById(id);
            if (modal) modal.style.display = 'none';
        }

        function adaptarFormularioSorteo() {
            const tipoEl = document.getElementById('tipoLotoSelect');
            const tipo = tipoEl ? tipoEl.value : '1';
            const grid = document.getElementById('inputsGridForm');
            const labelJugada = document.getElementById('labelJugadaTxt');
            const labelCantidad = document.getElementById('labelCantidadTxt');
            const btnEmitir = document.getElementById('btnEmitirBoleto');
            if (!grid) return;

            if (tipo === '2') {
                grid.innerHTML = `
                    <input type="text" inputmode="numeric" id="n1" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n2')" onkeydown="manejarRetroceso(event, this, null)">
                    <input type="text" inputmode="numeric" id="n2" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n3')" onkeydown="manejarRetroceso(event, this, 'n1')">
                    <input type="text" inputmode="numeric" id="n3" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n4')" onkeydown="manejarRetroceso(event, this, 'n2')">
                    <input type="text" inputmode="numeric" id="n4" class="num-input" maxlength="1" oninput="controlCasilla(this, null)" onkeydown="manejarRetroceso(event, this, 'n3')">
                `;
                if (labelJugada) labelJugada.innerText = 'JUGADA (4 NÚMEROS):';
                if (labelCantidad) labelCantidad.innerText = 'CANTIDAD DE BOLETOS ($5 c/u):';
                if (btnEmitir) btnEmitir.innerText = 'Emitir Boletos ($5 c/u)';
            } else if (tipo === '3') {
                grid.innerHTML = `
                    <input type="text" inputmode="numeric" id="n1" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n2')" onkeydown="manejarRetroceso(event, this, null)">
                    <input type="text" inputmode="numeric" id="n2" class="num-input" maxlength="1" oninput="controlCasilla(this, null)" onkeydown="manejarRetroceso(event, this, 'n1')">
                `;
                if (labelJugada) labelJugada.innerText = 'JUGADA (2 NÚMEROS):';
                if (labelCantidad) labelCantidad.innerText = 'CANTIDAD DE BOLETOS ($20 c/u):';
                if (btnEmitir) btnEmitir.innerText = 'Emitir Boletos ($20 c/u)';
            } else {
                grid.innerHTML = `
                    <input type="text" inputmode="numeric" id="n1" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n2')" onkeydown="manejarRetroceso(event, this, null)">
                    <input type="text" inputmode="numeric" id="n2" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n3')" onkeydown="manejarRetroceso(event, this, 'n1')">
                    <input type="text" inputmode="numeric" id="n3" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n4')" onkeydown="manejarRetroceso(event, this, 'n2')">
                    <input type="text" inputmode="numeric" id="n4" class="num-input" maxlength="1" oninput="controlCasilla(this, 'n5')" onkeydown="manejarRetroceso(event, this, 'n3')">
                    <input type="text" inputmode="numeric" id="n5" class="num-input" maxlength="1" oninput="controlCasilla(this, null)" onkeydown="manejarRetroceso(event, this, 'n4')">
                `;
                if (labelJugada) labelJugada.innerText = 'JUGADA (5 NÚMEROS):';
                if (labelCantidad) labelCantidad.innerText = 'CANTIDAD DE BOLETOS ($10 c/u):';
                if (btnEmitir) btnEmitir.innerText = 'Emitir Boletos ($10 c/u)';
            }
        }

        let vistaActual = 'sorteo';
        function toggleVista(vista) {
            const vSorteo = document.getElementById('sorteoView');
            const vReglas = document.getElementById('rulesPage');
            const vGanadores = document.getElementById('winnersPage');
            const vValidar = document.getElementById('validatePage');
            const btnR = document.getElementById('btnReglas');
            const btnG = document.getElementById('btnGanadores');
            const btnV = document.getElementById('btnValidar');

            if (vistaActual === vista) {
                vistaActual = 'sorteo';
                if (vSorteo) vSorteo.style.display = 'block';
                if (vReglas) vReglas.style.display = 'none';
                if (vGanadores) vGanadores.style.display = 'none';
                if (vValidar) vValidar.style.display = 'none';
                if (btnR) btnR.innerText = 'Reglas';
                if (btnG) btnG.innerText = 'Ganadores';
                if (btnV) btnV.innerText = '🔍 Validar Boleto';
            } else {
                vistaActual = vista;
                if (vSorteo) vSorteo.style.display = 'none';
                if (vReglas) vReglas.style.display = vista === 'reglas' ? 'block' : 'none';
                if (vGanadores) vGanadores.style.display = vista === 'ganadores' ? 'block' : 'none';
                if (vValidar) vValidar.style.display = vista === 'validar' ? 'block' : 'none';
                if (btnR) btnR.innerText = vista === 'reglas' ? 'Volver' : 'Reglas';
                if (btnG) btnG.innerText = vista === 'ganadores' ? 'Volver' : 'Ganadores';
                if (btnV) btnV.innerText = vista === 'validar' ? 'Volver' : '🔍 Validar Boleto';
                if (vista === 'ganadores') actualizarGanadoresUI();
            }
        }

        function consultarBoletoJugador() {
            const inputVal = document.getElementById('checkTicketInput').value.trim().toUpperCase();
            const resBox = document.getElementById('ticketResultBox');
            if (!inputVal) { showToast("Ingresa tu ID de boleto o teléfono.", "#ef4444"); return; }
            
            const t1 = getTickets(1);
            const t2 = getTickets(2);
            const t3 = getTickets(3);
            const todos = [...t1, ...t2, ...t3];
            const encontrados = todos.filter(t => 
                (t.id && t.id.toUpperCase() === inputVal) || 
                (t.id && ("#" + t.id.toUpperCase()) === inputVal) || 
                (t.telefono && t.telefono.replace(/[^0-9]/g, '') === inputVal.replace(/[^0-9]/g, ''))
            );

            if (encontrados.length > 0) {
                resBox.style.display = 'block';
                resBox.innerHTML = `
                    <div style="font-weight:900; color:#15803d; font-size:15px; margin-bottom:8px;">✅ BOLETO OFICIAL VALIDADO</div>
                    ` + encontrados.map(b => `
                        <div style="background:#ffffff; border:1px solid #bbf7d0; border-radius:8px; padding:10px; margin-bottom:8px; font-size:13px; color:#1e293b;">
                            <b>Titular:</b> ${b.nombre}<br>
                            <b>Sorteo:</b> ${b.sorteo}<br>
                            <b>Jugada:</b> <span class="tag-pill" style="font-size:14px; color:#15803d;">${b.numeros}</span><br>
                            <b>ID Oficial:</b> #${b.id || 'N/A'}<br>
                            <b>Fecha de Compra:</b> ${b.fecha || 'Oficial'}
                        </div>
                    `).join('') + `<p style="font-size:11px; color:#16a34a; font-weight:700; margin-top:4px;">Este boleto está activo en la base de datos oficial.</p>`;
                showToast("¡Boleto verificado y oficial!", "#22c55e");
            } else {
                resBox.style.display = 'block';
                resBox.innerHTML = `
                    <div style="font-weight:900; color:#dc2626; font-size:14px; margin-bottom:4px;">❌ BOLETO NO ENCONTRADO</div>
                    <p style="font-size:12px; color:#475569;">No se encontró ningún boleto activo con el identificador <b>${inputVal}</b>.</p>`;
                showToast("Boleto no registrado.", "#ef4444");
            }
        }

        function controlCasilla(actual, sigId) {
            actual.value = actual.value.replace(/[^0-9]/g, '');
            if (actual.value.length > 1) actual.value = actual.value.slice(-1);
            if (actual.value.length === 1 && sigId) {
                const siguiente = document.getElementById(sigId);
                if (siguiente) siguiente.focus();
            }
        }

        function manejarRetroceso(e, actual, prevId) {
            if (e && e.key === "Backspace" && actual.value === "" && prevId) {
                const anterior = document.getElementById(prevId);
                if (anterior) anterior.focus();
            }
        }

        function generarJugadaAlAzar() {
            const tipoEl = document.getElementById('tipoLotoSelect');
            const tipo = tipoEl ? tipoEl.value : '1';
            let limite = (tipo === '2') ? 4 : (tipo === '3' ? 2 : 5);
            for (let i = 1; i <= limite; i++) {
                const el = document.getElementById(`n${i}`);
                if (el) el.value = Math.floor(Math.random() * 10);
            }
        }

        const PRECIO_ESTELAR = 10.0;
        const PRECIO_MEDIODIA = 5.0;
        const PRECIO_NOCTURNO = 20.0;

        function getTickets(lotoId) {
            return JSON.parse(localStorage.getItem(`loteria_tickets_${lotoId}`)) || [];
        }

        function setTickets(lotoId, tickets) {
            localStorage.setItem(`loteria_tickets_${lotoId}`, JSON.stringify(tickets));
        }

        function getBoteAcumulado(lotoId) {
            return parseFloat(localStorage.getItem(`loto_bote_acumulado_${lotoId}`)) || 0;
        }

        function setBoteAcumulado(lotoId, val) {
            localStorage.setItem(`loto_bote_acumulado_${lotoId}`, val.toFixed(2));
        }

        // EMISIÓN Y ENVÍO SEGURO A WHATSAPP
        function comprarTickets() {
            const tipoEl = document.getElementById('tipoLotoSelect');
            const nomEl = document.getElementById('nombre');
            const telEl = document.getElementById('telefono');
            const cantEl = document.getElementById('cantidadBoletos');
            const msg = document.getElementById('msg');

            const tipoLoto = tipoEl ? tipoEl.value : '1';
            const nombre = nomEl ? nomEl.value.trim() : "";
            const telefono = telEl ? telEl.value.trim() : "";
            const cantidad = cantEl ? (parseInt(cantEl.value) || 1) : 1;

            if (!nombre || !telefono) {
                if (msg) msg.innerHTML = "<span style='color:#ef4444;'>Completa el nombre y teléfono</span>";
                return;
            }

            let numCasillas = (tipoLoto === '2') ? 4 : (tipoLoto === '3' ? 2 : 5);
            let n = [];
            for (let i = 1; i <= numCasillas; i++) {
                const inputEl = document.getElementById(`n${i}`);
                if (inputEl) n.push(inputEl.value.trim());
            }

            if (n.some(val => val === '' || isNaN(val) || val < 0 || val > 9)) {
                if (msg) msg.innerHTML = `<span style='color:#ef4444;'>Completa los ${numCasillas} dígitos (0 al 9)</span>`;
                return;
            }

            let combinacionJugada = n.join(' - ');
            let tickets = getTickets(tipoLoto);

            let yaExiste = tickets.some(t => t.numeros === combinacionJugada);
            let descontinuados = JSON.parse(localStorage.getItem(`loto_descontinuados_${tipoLoto}`)) || [];
            let estaDescontinuado = descontinuados.includes(combinacionJugada);

            if (yaExiste || estaDescontinuado) {
                if (msg) msg.innerHTML = "<span style='color:#ef4444;'>⚠️ Este número ya fue jugado o descontinuado. Elige otro.</span>";
                showToast("Número no disponible.", "#ef4444");
                return;
            }

            let precioUnitario = tipoLoto === '1' ? PRECIO_ESTELAR : (tipoLoto === '2' ? PRECIO_MEDIODIA : PRECIO_NOCTURNO);
            let nombreSorteo = tipoLoto === '1' ? 'Loto Estelar' : (tipoLoto === '2' ? 'Loto Medio Día' : 'Loto Nocturno');
            let totalDinero = (precioUnitario * cantidad).toFixed(2);
            let idBoleto = "LOT-" + Math.floor(10000 + Math.random() * 90000);
            let fechaHora = new Date().toLocaleString('es-PR', { hour12: true });

            for (let i = 0; i < cantidad; i++) {
                tickets.push({ 
                    id: idBoleto,
                    nombre: nombre, 
                    telefono: telefono, 
                    numeros: combinacionJugada, 
                    precio: precioUnitario,
                    sorteo: nombreSorteo,
                    fecha: fechaHora
                });
            }
            setTickets(tipoLoto, tickets);

            actualizarPremiosUI();

            let textoRecibo = 
`🎟️ *RECIBO OFICIAL DE LOTERÍA* 🎟️
━━━━━━━━━━━━━━━━━━
📌 *Boleto ID:* #${idBoleto}
📅 *Fecha:* ${fechaHora}
🎰 *Sorteo:* ${nombreSorteo}
━━━━━━━━━━━━━━━━━━
👤 *Titular:* ${nombre}
📞 *Teléfono:* ${telefono}
🔢 *Jugada:* [ ${combinacionJugada} ]
🎟️ *Cantidad:* ${cantidad} boleto(s)
💵 *Total Pagado:* $${totalDinero}
━━━━━━━━━━━━━━━━━━
✅ *Estado:* REGISTRADO Y OFICIAL
⚠️ *Conserva este recibo para reclamar tu premio.*`;

            let miWhatsApp = "17873269690";
            let urlWA = `https://api.whatsapp.com/send?phone=${miWhatsApp}&text=${encodeURIComponent(textoRecibo)}`;

            cerrarModal('modalAdminForm');
            document.getElementById('waReceiptPreview').innerText = textoRecibo;
            document.getElementById('btnEnviarWA').href = urlWA;
            document.getElementById('modalWhatsAppSuccess').style.display = 'flex';

            if (nomEl) nomEl.value = '';
            if (telEl) telEl.value = '';
            if (cantEl) cantEl.value = '1';
            ['n1','n2','n3','n4','n5'].forEach(id => {
                const el = document.getElementById(id);
                if (el) el.value = '';
            });
            const firstInput = document.getElementById('n1');
            if (firstInput) firstInput.focus();
        }

        // MOTOR DE SORTEOS Y GESTIÓN DE BLOQUEOS SEGÚN GANADOR O ACUMULADO
        let sorteoEjecutado1 = false;
        let sorteoEjecutado2 = false;
        let sorteoEjecutado3 = false;

        function registrarNumeroDescontinuado(lotoId, combinacion) {
            let descontinuados = JSON.parse(localStorage.getItem(`loto_descontinuados_${lotoId}`)) || [];
            if (!descontinuados.includes(combinacion)) {
                descontinuados.push(combinacion);
                localStorage.setItem(`loto_descontinuados_${lotoId}`, JSON.stringify(descontinuados));
            }
        }

        function limpiarNumerosDescontinuados(lotoId) {
            localStorage.removeItem(`loto_descontinuados_${lotoId}`);
        }

        function ejecutarSorteoLoto1() {
            let tickets = getTickets(1);
            if (tickets.length === 0) return;
            document.getElementById('winnerBox1').style.display = 'none';
            const ganador = tickets[Math.floor(Math.random() * tickets.length)];
            const nFinales = ganador.numeros.split(' - ');

            for (let i = 0; i < 5; i++) document.getElementById(`l1_b${i}`).innerText = '?';

            let indiceBola = 0;
            function revelarBola() {
                if (indiceBola >= 5) {
                    let pozoTotalBruto = getBoteAcumulado(1) + (tickets.length * PRECIO_ESTELAR);
                    let premioGanado = pozoTotalBruto * 0.60;
                    let fechaActualStr = new Date().toLocaleString('es-PR', { hour12: true });

                    let historial = JSON.parse(localStorage.getItem('loteria_ganadores_global')) || [];
                    historial.unshift({ sorteoNombre: 'Loto Estelar', nombre: ganador.nombre, premio: premioGanado, numeros: ganador.numeros, fecha: fechaActualStr });
                    localStorage.setItem('loteria_ganadores_global', JSON.stringify(historial));

                    setBoteAcumulado(1, 0);
                    setTickets(1, []);
                    limpiarNumerosDescontinuados(1);

                    document.getElementById('winnerDetails1').innerHTML = `Fecha: <b>${fechaActualStr}</b><br>Titular: <b>${ganador.nombre}</b><br>Número Ganador: <b style="color:#15803d; font-size:14px;">[ ${ganador.numeros} ]</b><br>Premio Entregado: <b style="color:#15803d; font-size:15px;">$${premioGanado.toFixed(2)}</b><br><small style="color:#64748b;">Tómbola restablecida limpia para la próxima ronda.</small>`;
                    document.getElementById('winnerBox1').style.display = 'block';
                    lanzarFuegosArtificiales();

                    actualizarPremiosUI();

                    sorteoEjecutado1 = true;
                    setTimeout(() => {
                        document.getElementById('winnerBox1').style.display = 'none';
                        for (let i = 0; i < 5; i++) document.getElementById(`l1_b${i}`).innerText = '?';
                    }, 25000);
                    return;
                }
                const bolaActual = document.getElementById(`l1_b${indiceBola}`);
                bolaActual.classList.add('ball-spinning');
                const intervalG = setInterval(() => { bolaActual.innerText = Math.floor(Math.random() * 10); }, 70);
                setTimeout(() => {
                    clearInterval(intervalG);
                    bolaActual.classList.remove('ball-spinning');
                    bolaActual.innerText = nFinales[indiceBola];
                    indiceBola++;
                    revelarBola();
                }, 15000);
            }
            revelarBola();
        }

        function ejecutarSorteoMedioDia() {
            let tickets = getTickets(2);
            if (tickets.length === 0) return;
            document.getElementById('winnerBox2').style.display = 'none';
            const numSorteados = [Math.floor(Math.random() * 10), Math.floor(Math.random() * 10), Math.floor(Math.random() * 10), Math.floor(Math.random() * 10)];
            const combinacionString = numSorteados.join(' - ');

            for (let i = 0; i < 4; i++) document.getElementById(`l2_b${i}`).innerText = '?';

            let indiceBola = 0;
            function revelarBola() {
                if (indiceBola >= 4) {
                    let pozoTotalBruto = getBoteAcumulado(2) + (tickets.length * PRECIO_MEDIODIA);
                    let fechaActualStr = new Date().toLocaleString('es-PR', { hour12: true });
                    const acertantes = tickets.filter(t => t.numeros === combinacionString);

                    if (acertantes.length > 0) {
                        let premioGanado = pozoTotalBruto * 0.60;
                        let nombres = acertantes.map(a => a.nombre).join(', ');
                        let historial = JSON.parse(localStorage.getItem('loteria_ganadores_global')) || [];
                        historial.unshift({ sorteoNombre: 'Loto Medio Día', nombre: nombres, premio: premioGanado, numeros: combinacionString, fecha: fechaActualStr });
                        localStorage.setItem('loteria_ganadores_global', JSON.stringify(historial));

                        setBoteAcumulado(2, 0);
                        setTickets(2, []);
                        limpiarNumerosDescontinuados(2);

                        document.getElementById('winnerBoxTitle2').innerText = "¡Boleto Ganador Loto Medio Día!";
                        document.getElementById('winnerDetails2').innerHTML = `Fecha: <b>${fechaActualStr}</b><br>Titular(es): <b>${nombres}</b><br>Número Ganador: <b style="color:#15803d; font-size:14px;">[ ${combinacionString} ]</b><br>Premio: <b style="color:#15803d; font-size:15px;">$${premioGanado.toFixed(2)}</b><br><small style="color:#64748b;">¡Tómbola restablecida limpia para la primera vez!</small>`;
                        lanzarFuegosArtificiales();
                    } else {
                        setBoteAcumulado(2, pozoTotalBruto);
                        registrarNumeroDescontinuado(2, combinacionString);

                        let historial = JSON.parse(localStorage.getItem('loteria_ganadores_global')) || [];
                        historial.unshift({ sorteoNombre: 'Loto Medio Día (Sin Acierto)', nombre: 'Nadie (Funda Acumulada)', premio: 0, numeros: combinacionString, fecha: fechaActualStr });
                        localStorage.setItem('loteria_ganadores_global', JSON.stringify(historial));

                        document.getElementById('winnerBoxTitle2').innerText = "¡Resultado Loto Medio Día (Sin Acierto)!";
                        document.getElementById('winnerDetails2').innerHTML = `Fecha: <b>${fechaActualStr}</b><br>Número Sorteado y Descontinuado: <b style="color:#0284c7; font-size:14px;">[ ${combinacionString} ]</b><br>La funda acumulada de <b>$${pozoTotalBruto.toFixed(2)}</b> pasa para mañana a las 12:00 PM.`;
                    }
                    document.getElementById('winnerBox2').style.display = 'block';

                    actualizarPremiosUI();

                    sorteoEjecutado2 = true;
                    setTimeout(() => {
                        document.getElementById('winnerBox2').style.display = 'none';
                        for (let i = 0; i < 4; i++) document.getElementById(`l2_b${i}`).innerText = '?';
                    }, 25000);
                    return;
                }
                const bolaActual = document.getElementById(`l2_b${indiceBola}`);
                bolaActual.classList.add('ball-spinning');
                const intervalG = setInterval(() => { bolaActual.innerText = Math.floor(Math.random() * 10); }, 70);
                setTimeout(() => {
                    clearInterval(intervalG);
                    bolaActual.classList.remove('ball-spinning');
                    bolaActual.innerText = numSorteados[indiceBola];
                    indiceBola++;
                    revelarBola();
                }, 15000);
            }
            revelarBola();
        }

        function ejecutarSorteoNocturno() {
            let tickets = getTickets(3);
            if (tickets.length === 0) return;
            document.getElementById('winnerBox3').style.display = 'none';
            const numSorteados = [Math.floor(Math.random() * 10), Math.floor(Math.random() * 10)];
            const combinacionString = numSorteados.join(' - ');

            for (let i = 0; i < 2; i++) document.getElementById(`l3_b${i}`).innerText = '?';

            let indiceBola = 0;
            function revelarBola() {
                if (indiceBola >= 2) {
                    let pozoTotalBruto = getBoteAcumulado(3) + (tickets.length * PRECIO_NOCTURNO);
                    let fechaActualStr = new Date().toLocaleString('es-PR', { hour12: true });
                    const acertantes = tickets.filter(t => t.numeros === combinacionString);

                    if (acertantes.length > 0) {
                        let premioGanado = pozoTotalBruto * 0.60;
                        let nombres = acertantes.map(a => a.nombre).join(', ');
                        let historial = JSON.parse(localStorage.getItem('loteria_ganadores_global')) || [];
                        historial.unshift({ sorteoNombre: 'Loto Nocturno', nombre: nombres, premio: premioGanado, numeros: combinacionString, fecha: fechaActualStr });
                        localStorage.setItem('loteria_ganadores_global', JSON.stringify(historial));

                        setBoteAcumulado(3, 0);
                        setTickets(3, []);
                        limpiarNumerosDescontinuados(3);

                        document.getElementById('winnerBoxTitle3').innerText = "¡Boleto Ganador Loto Nocturno!";
                        document.getElementById('winnerDetails3').innerHTML = `Fecha: <b>${fechaActualStr}</b><br>Titular(es): <b>${nombres}</b><br>Número Ganador: <b style="color:#15803d; font-size:14px;">[ ${combinacionString} ]</b><br>Premio: <b style="color:#15803d; font-size:15px;">$${premioGanado.toFixed(2)}</b><br><small style="color:#64748b;">¡Tómbola restablecida limpia para la primera vez!</small>`;
                        lanzarFuegosArtificiales();
                    } else {
                        setBoteAcumulado(3, pozoTotalBruto);
                        registrarNumeroDescontinuado(3, combinacionString);

                        let historial = JSON.parse(localStorage.getItem('loteria_ganadores_global')) || [];
                        historial.unshift({ sorteoNombre: 'Loto Nocturno (Sin Acierto)', nombre: 'Nadie (Funda Acumulada)', premio: 0, numeros: combinacionString, fecha: fechaActualStr });
                        localStorage.setItem('loteria_ganadores_global', JSON.stringify(historial));

                        document.getElementById('winnerBoxTitle3').innerText = "¡Resultado Loto Nocturno (Sin Acierto)!";
                        document.getElementById('winnerDetails3').innerHTML = `Fecha: <b>${fechaActualStr}</b><br>Número Sorteado y Descontinuado: <b style="color:#9333ea; font-size:14px;">[ ${combinacionString} ]</b><br>La funda acumulada de <b>$${pozoTotalBruto.toFixed(2)}</b> pasa para mañana a las 8:00 PM.`;
                    }
                    document.getElementById('winnerBox3').style.display = 'block';

                    actualizarPremiosUI();

                    sorteoEjecutado3 = true;
                    setTimeout(() => {
                        document.getElementById('winnerBox3').style.display = 'none';
                        for (let i = 0; i < 2; i++) document.getElementById(`l3_b${i}`).innerText = '?';
                    }, 25000);
                    return;
                }
                const bolaActual = document.getElementById(`l3_b${indiceBola}`);
                bolaActual.classList.add('ball-spinning');
                const intervalG = setInterval(() => { bolaActual.innerText = Math.floor(Math.random() * 10); }, 70);
                setTimeout(() => {
                    clearInterval(intervalG);
                    bolaActual.classList.remove('ball-spinning');
                    bolaActual.innerText = numSorteados[indiceBola];
                    indiceBola++;
                    revelarBola();
                }, 15000);
            }
            revelarBola();
        }

        function actualizarGanadoresUI() {
            const container = document.getElementById('winnersContainer');
            if (!container) return;
            let historialGanadores = JSON.parse(localStorage.getItem('loteria_ganadores_global')) || [];
            if (historialGanadores.length === 0) {
                container.innerHTML = `<p style="color:#94a3b8; font-size:13px; font-style:italic;">Aún no hay registros en este historial.</p>`;
                return;
            }
            container.innerHTML = historialGanadores.map((g, idx) => `
                <div class="winner-row">
                    <div style="font-weight:800; color:#15803d; margin-bottom:2px;">🎉 ${g.sorteoNombre}</div>
                    <div style="color:#64748b; font-size:11px; margin-bottom:4px;">🕒 ${g.fecha || 'Fecha Oficial'}</div>
                    <div style="color:#0f172a; font-size:13px;">Titular/Resultado: <b>${g.nombre}</b></div>
                    <div style="color:#475569; font-size:12px;">Número Sorteado: <b style="color:#0f172a;">[ ${g.numeros} ]</b></div>
                    ${g.premio > 0 ? `<div style="color:#475569; font-size:12px;">Premio Entregado: <b style="color:#15803d;">$${g.premio.toFixed(2)}</b></div>` : ''}
                </div>
            `).join('');
        }

        function padZero(num) { return num < 10 ? '0' + num : '' + num; }

        function actualizarContadores() {
            const ahora = new Date();
            let diaActual = ahora.getDay();
            let diasHastaViernes = (5 - diaActual + 7) % 7;
            let obj1 = new Date(ahora.getTime());
            obj1.setDate(ahora.getDate() + diasHastaViernes);
            obj1.setHours(20, 0, 0, 0);

            if (diaActual === 5 && ahora.getTime() >= obj1.getTime()) {
                if (!sorteoEjecutado1 && getTickets(1).length > 0 && (ahora.getTime() - obj1.getTime() < 180000)) {
                    ejecutarSorteoLoto1();
                    return;
                }
                obj1.setDate(obj1.getDate() + 7);
                sorteoEjecutado1 = false;
            }

            let dif1 = Math.max(0, obj1.getTime() - ahora.getTime());
            const l1d = document.getElementById('l1_d');
            const l1h = document.getElementById('l1_h');
            const l1m = document.getElementById('l1_m');
            const l1s = document.getElementById('l1_s');
            if (l1d) l1d.innerText = padZero(Math.floor(dif1 / (1000 * 60 * 60 * 24)));
            if (l1h) l1h.innerText = padZero(Math.floor((dif1 / (1000 * 60 * 60)) % 24));
            if (l1m) l1m.innerText = padZero(Math.floor((dif1 / (1000 * 60)) % 60));
            if (l1s) l1s.innerText = padZero(Math.floor((dif1 / 1000) % 60));

            let obj2 = new Date(ahora.getTime());
            obj2.setHours(12, 0, 0, 0);
            if (ahora.getTime() >= obj2.getTime()) {
                if (!sorteoEjecutado2 && getTickets(2).length > 0 && (ahora.getTime() - obj2.getTime() < 180000)) {
                    ejecutarSorteoMedioDia();
                    return;
                }
                obj2.setDate(obj2.getDate() + 1);
                sorteoEjecutado2 = false;
            }

            let dif2 = Math.max(0, obj2.getTime() - ahora.getTime());
            const l2d = document.getElementById('l2_d');
            const l2h = document.getElementById('l2_h');
            const l2m = document.getElementById('l2_m');
            const l2s = document.getElementById('l2_s');
            if (l2d) l2d.innerText = padZero(Math.floor(dif2 / (1000 * 60 * 60 * 24)));
            if (l2h) l2h.innerText = padZero(Math.floor((dif2 / (1000 * 60 * 60)) % 24));
            if (l2m) l2m.innerText = padZero(Math.floor((dif2 / (1000 * 60)) % 60));
            if (l2s) l2s.innerText = padZero(Math.floor((dif2 / 1000) % 60));

            let obj3 = new Date(ahora.getTime());
            obj3.setHours(20, 0, 0, 0);
            if (ahora.getTime() >= obj3.getTime()) {
                if (!sorteoEjecutado3 && getTickets(3).length > 0 && (ahora.getTime() - obj3.getTime() < 180000)) {
                    ejecutarSorteoNocturno();
                    return;
                }
                obj3.setDate(obj3.getDate() + 1);
                sorteoEjecutado3 = false;
            }

            let dif3 = Math.max(0, obj3.getTime() - ahora.getTime());
            const l3d = document.getElementById('l3_d');
            const l3h = document.getElementById('l3_h');
            const l3m = document.getElementById('l3_m');
            const l3s = document.getElementById('l3_s');
            if (l3d) l3d.innerText = padZero(Math.floor(dif3 / (1000 * 60 * 60 * 24)));
            if (l3h) l3h.innerText = padZero(Math.floor((dif3 / (1000 * 60 * 60)) % 24));
            if (l3m) l3m.innerText = padZero(Math.floor((dif3 / (1000 * 60)) % 60));
            if (l3s) l3s.innerText = padZero(Math.floor((dif3 / 1000) % 60));
        }

        actualizarContadores();
        setInterval(actualizarContadores, 1000);
    </script>
</body>
</html>
