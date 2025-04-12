# buscaminas_flutter3 
import 'package:flutter/material.dart';
import 'dart:math';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Buscaminas',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: BuscaminasGame(),
    );
  }
}

class BuscaminasGame extends StatefulWidget {
  @override
  _BuscaminasGameState createState() => _BuscaminasGameState();
}

class _BuscaminasGameState extends State<BuscaminasGame> {
  late List<List<int>> _tablero;
  late List<List<bool>> _revelado;
  late List<List<bool>> _bandera;
  final int filas = 9;
  final int columnas = 9;
  final int minas = 10;

  @override
  void initState() {
    super.initState();
    _iniciarJuego();
  }

  // Inicia el juego y coloca las minas
  void _iniciarJuego() {
    _tablero = List.generate(filas, (_) => List.generate(columnas, (_) => 0));
    _revelado = List.generate(filas, (_) => List.generate(columnas, (_) => false));
    _bandera = List.generate(filas, (_) => List.generate(columnas, (_) => false));

    Random rand = Random();
    int minasColocadas = 0;
    while (minasColocadas < minas) {
      int fila = rand.nextInt(filas);
      int columna = rand.nextInt(columnas);

      if (_tablero[fila][columna] == 0) {
        _tablero[fila][columna] = -1; // Coloca mina
        minasColocadas++;

        // Actualiza las casillas adyacentes
        for (int i = -1; i <= 1; i++) {
          for (int j = -1; j <= 1; j++) {
            if (fila + i >= 0 && fila + i < filas && columna + j >= 0 && columna + j < columnas) {
              if (_tablero[fila + i][columna + j] != -1) {
                _tablero[fila + i][columna + j]++;
              }
            }
          }
        }
      }
    }
  }

  // Revela una casilla
  void _revelar(int fila, int columna) {
    if (_revelado[fila][columna] || _bandera[fila][columna]) return;

    setState(() {
      _revelado[fila][columna] = true;
    });

    if (_tablero[fila][columna] == -1) {
      // Ha tocado una mina
      _mostrarFinJuego(false);
    } else if (_tablero[fila][columna] == 0) {
      // Casilla vacía, revelar adyacentes
      for (int i = -1; i <= 1; i++) {
        for (int j = -1; j <= 1; j++) {
          if (fila + i >= 0 && fila + i < filas && columna + j >= 0 && columna + j < columnas) {
            if (!_revelado[fila + i][columna + j]) {
              _revelar(fila + i, columna + j);
            }
          }
        }
      }
    }
  }

  // Marca o desmarca bandera
  void _marcarBandera(int fila, int columna) {
    setState(() {
      _bandera[fila][columna] = !_bandera[fila][columna];
    });
  }

  // Muestra el fin del juego
  void _mostrarFinJuego(bool ganado) {
    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text(ganado ? '¡Ganaste!' : 'Perdiste'),
          content: Text('¡El juego ha terminado!'),
          actions: <Widget>[
            TextButton(
              onPressed: () {
                setState(() {
                  _iniciarJuego();
                });
                Navigator.of(context).pop();
              },
              child: Text('Reiniciar'),
            ),
          ],
        );
      },
    );
  }

  // Construye la interfaz del tablero
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Buscaminas')),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          for (int i = 0; i < filas; i++)
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                for (int j = 0; j < columnas; j++)
                  GestureDetector(
                    onTap: () => _revelar(i, j),
                    onLongPress: () => _marcarBandera(i, j),
                    child: Container(
                      margin: EdgeInsets.all(2),
                      width: 30,
                      height: 30,
                      decoration: BoxDecoration(
                        color: _revelado[i][j]
                            ? (_tablero[i][j] == -1 ? Colors.red : Colors.grey)
                            : Colors.blue,
                        border: Border.all(color: Colors.black),
                      ),
                      child: Center(
                        child: _revelado[i][j]
                            ? (_tablero[i][j] == -1
                                ? Icon(Icons.brightness_1, color: Colors.white, size: 18)
                                : Text(
                                    _tablero[i][j] == 0 ? '' : _tablero[i][j].toString(),
                                    style: TextStyle(color: Colors.white),
                                  ))
                            : _bandera[i][j]
                                ? Icon(Icons.flag, color: Colors.white, size: 18)
                                : null,
                      ),
                    ),
                  ),
              ],
            ),
        ],
      ),
    );
  }
}
