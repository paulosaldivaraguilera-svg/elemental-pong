# ELEMENTAL PONG: Chrono & Chaos
## Game Design Document Técnico v2.0

*Videojuego minimalista con mecánicas elementales y control del tiempo*

---

## Estado del Prototype v2.0

| Feature | Estado | Descripción |
|---------|--------|-------------|
| **Menú Principal** | ✅ | UI con título, botón jugar, récord |
| **Sistema de Niveles** | ✅ | Waves que aumentan dificultad |
| **6 Elementos** | ✅ | Fuego, Hielo, Rayo, Tierra, Aire, Agua |
| **Chrono-Break** | ✅ | Ralentizar tiempo 2s |
| **Partículas** | ✅ | Efectos visuales |
| **Pantalla Game Over** | ✅ | Con récord localStorage |
| **Streak System** | ✅ | Bonificaciones por reflejos |
| **Screen Shake** | ✅ | Feedback al recibir punto |

---

## Controles

| Tecla | Acción |
|-------|--------|
| **← → / A D** | Mover paleta |
| **ESPACIO** | Cargar poder (100% = elemento aleatorio) |
| **C** | Chrono-Break (ralentiza tiempo 2s) |

---

## Sistema de Elementos

| Elemento | Efecto | Color |
|----------|--------|-------|
| **Fuego** | Velocidad x2 + partículas | 🔴 Rojo |
| **Hielo** | Efecto slippery | 🔵 Cyan |
| **Rayo** | Trayectoria errática | 🟡 Amarillo |
| **Tierra** | Velocidad reducida (bola pesada) | 🟤 Marrón |
| **Aire** | Curva sinusoidal | ⚪ Celeste |
| **Agua** | Rebotes erráticos en paredes | 🔵 Azul |

---

## Sistema de Progresión

- **Waves:** Cada 5 puntos, aumenta nivel
- **Enemigo:** Más rápido en waves altos
- **Streak:** Golpes perfectos = más Chrono

---

## Deploy

**Prototype:** https://paulosaldivaraguilera-svg.github.io/elemental-pong/prototype.html

---

## Desarrollado en Unity (Próxima Fase)

Ver `index.html` para el GDD completo de Unity.
