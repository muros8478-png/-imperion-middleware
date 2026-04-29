"""
🚀 SERVIDOR MIDDLEWARE - A & S IMPERION E.I.R.L.
Versión Railway - Conexión OSCE
"""

from flask import Flask, jsonify
import requests
import ssl
from datetime import datetime, timedelta

app = Flask(__name__)

# Configurar SSL
ssl._create_default_https_context = ssl._create_unverified_context

@app.route('/')
def inicio():
    return jsonify({
        'servidor': 'IMPERION-BOT Middleware',
        'version': '1.0',
        'estado': 'activo',
        'empresa': 'A & S IMPERION E.I.R.L.'
    })

@app.route('/licitaciones')
def obtener_licitaciones():
    try:
        print(f"[{datetime.now().strftime('%H:%M:%S')}] 📡 Consultando OSCE...")
        
        hoy = datetime.now()
        ayer = hoy - timedelta(hours=24)
        
        url_osce = (
            'https://contratacionesabiertas.osce.gob.pe/api/releases/'
            f'?dateFrom={ayer.strftime("%Y-%m-%d")}'
            f'&dateTo={hoy.strftime("%Y-%m-%d")}'
            f'&page_size=50'
        )
        
        print(f"   URL: {url_osce}")
        
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
            'Accept': 'application/json'
        }
        
        respuesta = requests.get(url_osce, headers=headers, timeout=30)
        datos = respuesta.json()
        
        resultados = []
        releases = datos.get('results', datos.get('releases', []))
        
        for paquete in releases:
            items = paquete.get('releases', [paquete])
            for release in items:
                try:
                    tender = release.get('tender', {})
                    if not tender.get('title'):
                        continue
                    
                    entidad = 'No especificada'
                    for party in release.get('parties', []):
                        if 'buyer' in party.get('roles', []):
                            entidad = party.get('name', entidad)
                            break
                    
                    region = 'NACIONAL'
                    for party in release.get('parties', []):
                        if party.get('address', {}).get('region'):
                            region = party['address']['region']
                            break
                    
                    monto = 0
                    valor = tender.get('value', {})
                    if valor.get('amount'):
                        monto = float(valor['amount'])
                    
                    resultados.append({
                        'titulo': tender.get('title', ''),
                        'entidad': entidad,
                        'region': region,
                        'rubro': tender.get('items', [{}])[0].get('classification', {}).get('description', ''),
                        'monto': monto,
                        'fechaCierre': tender.get('tenderPeriod', {}).get('endDate', ''),
                        'plazo': '',
                        'tdr': (tender.get('description', '') or '')[:300]
                    })
                except Exception as e:
                    print(f"   ⚠️ Error: {e}")
        
        print(f"   ✅ {len(resultados)} licitaciones")
        
        return jsonify({
            'exito': True,
            'total': len(resultados),
            'licitaciones': resultados
        })
        
    except Exception as e:
        print(f"   ❌ Error: {e}")
        return jsonify({
            'exito': False,
            'error': str(e)
        })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
