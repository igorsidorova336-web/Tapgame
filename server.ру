from flask import Flask, jsonify, request
from flask_cors import CORS
from datetime import datetime

app = Flask(__name__)
CORS(app)

active_players = {}

@app.route('/')
def index():
    return jsonify({'status': 'ok', 'message': 'Сервер работает!'})

@app.route('/api/online', methods=['GET'])
def get_online():
    now = datetime.now()
    to_remove = []
    for pid, last_seen in active_players.items():
        try:
            last = datetime.fromisoformat(last_seen)
            if (now - last).total_seconds() > 15:
                to_remove.append(pid)
        except:
            to_remove.append(pid)
    for pid in to_remove:
        del active_players[pid]
    return jsonify({'online': len(active_players)})

@app.route('/api/online/join', methods=['POST'])
def join():
    data = request.json
    player_id = data.get('player_id')
    if not player_id:
        return jsonify({'error': 'player_id required'}), 400
    active_players[player_id] = datetime.now().isoformat()
    return jsonify({'status': 'ok', 'online': len(active_players)})

@app.route('/api/online/ping', methods=['POST'])
def ping():
    data = request.json
    player_id = data.get('player_id')
    if player_id:
        active_players[player_id] = datetime.now().isoformat()
    return jsonify({'status': 'ok', 'online': len(active_players)})

@app.route('/api/online/leave', methods=['POST'])
def leave():
    data = request.json
    player_id = data.get('player_id')
    if player_id and player_id in active_players:
        del active_players[player_id]
    return jsonify({'status': 'ok', 'online': len(active_players)})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=10000)
