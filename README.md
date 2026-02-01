# ELEMENTAL PONG: Chrono & Chaos
## Game Design Document Técnico

*Videojuego minimalista con mecánicas elementales y control del tiempo*

---

## Plataformas

| Plataforma | Target | Controles |
|------------|--------|-----------|
| **Mobile (iOS/Android)** | Casual gamers | Touch screen |
| **Nintendo Switch** | Core gamers | Joy-Cons, Dock mode |

---

## Storyboard de Progresión

### Fase 1: "El Vació" (Tutorial)
- **Visual:** Blanco y negro puro
- **Mecánica:** Sin poderes, solo física Pong básica
- **Objetivo:** Aprender ritmo, fills barra de Chrono con devoluciones precisas
- **Duración:** Infinite (arcade mode)

### Fase 2: "El Despertar Ígneo"
- **Visual:** Tonos rojos y naranjas
- **Mecánica:** IA empieza a usar Fuego
- **Objetivo:** Sobrevivir 60 segundos O anotar 5 puntos
- **Recompensa:** Desbloquear poder de Fuego

### Fase 3: "Boss Elemental"
- **Visual:** Transiciones de color según elemento activo
- **Mecánica:** Enemigo "Barra Gigante" que ocupa mitad de pantalla
- **Objetivo:** Usar Fuego (contraelemento) para derretir defensa
- **Obstáculos:** Introducción de bloques en la cancha

---

## Controles

### Mobile (Touch)
| Gestos | Acción |
|--------|--------|
| **Pantalla dividida invisible** | Mover paleta (izquierda/derecha) |
| **Deslizar hacia abajo** | Cargar poder elemental |
| **Doble tap rápido** | Activar Chrono-Break (ralentizar tiempo) |

### Nintendo Switch
| Control | Acción |
|---------|--------|
| **Stick Izquierdo** | Movimiento de paleta |
| **Gatillo R (ZR)** | Activar poder elemental cargado |
| **Gatillo L (ZL)** | Chrono-Break |
| **Botón A** | Golpe ofensivo (si hay carga) |

---

## Sistema Elemental

### Los 6 Elementos

| Elemento | Color | Efecto Principal | Contra-elemento | Estrategia |
|----------|-------|------------------|-----------------|------------|
| **Fuego (Ignis)** | Rojo | Velocidad +80%, rastro visual | Hielo | Tiros rápidos |
| **Hielo (Glacies)** | Cyan | Fricción reducidas (oponente resbala) | Fuego | Descolocar rival |
| **Rayo (Fulgur)** | Amarillo | Trayectoria Zig-Zag | Tierra | Impredecible |
| **Tierra (Terra)** | Marrón | Peso extra, empuja paleta rival | Aire | Control territorial |
| **Aire (Ventus)** | Celeste | Efecto curvo (Aftertouch) | - | Tiros con curva |
| **Agua (Aqua)** | Azul | Rebote errático, ondas visuales | - | Desorientar |

### Carga de Poderes
- **Perfect Parry (Timing exacto):** +25% carga
- **Golpe ofensivo:** +50% carga
- **Anotar punto:** +15% carga
- **Recibir gol:** -20% carga (recupera con puntos)

### Chrono-Break
- **Activación:** Doble tap / Gatillo L
- **Efecto:** Time.timeScale = 0.1x por 2 segundos
- **Costo:** Barra completa
- **Usos estratégicos:** Reposicionar para saves imposibles / Preparar tiro elemental

---

## Arquitectura Unity

### Estructura de Archivos
```
Assets/
├── _Core/
│   ├── Scripts/
│   │   ├── GameManager.cs
│   │   ├── TimeManager.cs
│   │   └── AudioManager.cs
│   └── Prefabs/
│       ├── Ball.prefab
│       ├── Paddle.prefab
│       └── Wall.prefab
├── Entities/
│   ├── Ball/
│   │   ├── BallController.cs
│   │   └── ElementEffects/
│   │       ├── ElementData.cs (SO)
│   │       ├── ElementalEffectBase.cs
│   │       ├── FireEffect.cs
│   │       ├── IceEffect.cs
│   │       ├── ThunderEffect.cs
│   │       ├── EarthEffect.cs
│   │       ├── AirEffect.cs
│   │       └── WaterEffect.cs
│   └── Paddle/
│       └── PaddleController.cs
├── UI/
│   ├── GameHUD.cs
│   ├── ChargeBar.cs
│   └── ScoreDisplay.cs
└── Systems/
    ├── Input/
    │   ├── InputManager.cs
    │   ├── TouchInput.cs
    │   └── JoyconInput.cs
    └── Physics/
        ├── CollisionHandler.cs
        └── ScreenShake.cs
```

### Scripts Clave

#### GameManager.cs
```csharp
public class GameManager : MonoBehaviour {
    public enum GameState { Menu, Playing, Paused, GameOver }
    public GameState currentState;
    public int playerScore, enemyScore;
    
    public event Action OnScore;
    public event Action OnGameOver;
    
    public void AddPoint(bool isPlayer) {
        if(isPlayer) playerScore++;
        else enemyScore++;
        OnScore?.Invoke();
        CheckWinCondition();
    }
}
```

#### BallController.cs
```csharp
public class BallController : MonoBehaviour {
    public Rigidbody2D rb;
    public ElementData currentElement;
    private float baseSpeed = 10f;
    
    public void ApplyEffect(ElementData element) {
        currentElement = element;
        // Trigger particle effect, modify physics
        rb.velocity = rb.velocity.normalized * baseSpeed * element.speedMultiplier;
    }
    
    void OnCollisionEnter2D(Collision2D collision) {
        if(collision.gameObject.CompareTag("Paddle")) {
            float hitPoint = CalculateHitPoint(collision.transform.position);
            // Apply Aftertouch if Air element
            if(currentElement.elementType == ElementType.Air) {
                ApplyAftertouch();
            }
        }
    }
}
```

#### ElementData.cs (ScriptableObject)
```csharp
[CreateAssetMenu(fileName = "ElementData", menuName = "Elemental/Data")]
public class ElementData : ScriptableObject {
    public string elementName;
    public ElementType elementType;
    public Color color;
    public float speedMultiplier = 1f;
    public float frictionModifier = 1f;
    public GameObject particlePrefab;
    public AudioClip activationSound;
}
```

#### TimeManager.cs
```csharp
public class TimeManager : MonoBehaviour {
    private float originalTimeScale;
    private Coroutine chronoRoutine;
    
    public void ActivateChronoBreak(float duration) {
        if(chronoRoutine != null) StopCoroutine(chronoRoutine);
        chronoRoutine = StartCoroutine(ChronoBreakRoutine(duration));
    }
    
    private IEnumerator ChronoBreakRoutine(float duration) {
        originalTimeScale = Time.timeScale;
        Time.timeScale = 0.1f;
        // Visual feedback: grayscale, slow particles
        yield return new WaitForSecondsRealtime(duration);
        Time.timeScale = Mathf.Lerp(0.1f, 1f, 1f); // Smooth return
    }
}
```

#### PaddleController.cs
```csharp
public class PaddleController : MonoBehaviour {
    public float moveSpeed = 15f;
    public float friction = 1f; // Modificado por Hielo
    
    void FixedUpdate() {
        float input = GetInputAxis();
        // Ice element reduces friction temporarily
        float actualSpeed = moveSpeed * (1f / friction);
        rb.MovePosition(rb.position + Vector2.right * input * actualSpeed * Time.fixedDeltaTime);
        // Clamp to screen bounds
    }
}
```

---

## Assets Requeridos

### Gráficos
| Asset | Descripción | Formato |
|-------|-------------|---------|
| **Ball** | Pelota circular con glow | PNG (32x32) |
| **Paddle** | Paleta rectangular | PNG (8x64) |
| **Particles** | 6 efectos elementales | PNG (16x16) loop |
| **Background** | Grid minimalista | PNG (1920x1080) |
| **UI Icons** | 6 iconos elementales | SVG → PNG |

### Audio
| Asset | Descripción | Formato |
|-------|-------------|---------|
| **Hit** | Sonido de impacto | WAV 44.1kHz |
| **Element Activate** | 6 variaciones | WAV |
| **Chrono** | Whoosh temporal | WAV |
| **Music** | Loop ambiental minimalista | MP3 128kbps |

---

## Fases de Desarrollo

### Phase 1: Core (MVP)
- [ ] Movimiento de paleta básico
- [ ] Rebote de pelota
- [ ] Score system
- [ ] Pantalla de Game Over
- [ ] Menú principal

### Phase 2: Elementos
- [ ] Sistema de ScriptableObjects
- [ ] Implementar Fuego (velocidad +)
- [ ] Implementar Hielo (fricción -)
- [ ] Implementar Rayo (Zig-Zag)
- [ ] Implementar Tierra (peso +)
- [ ] Implementar Aire (curva)
- [ ] Implementar Agua (rebote errático)

### Phase 3: Chrono
- [ ] TimeManager
- [ ] Barra de Chrono UI
- [ ] Doble tap / Gatillo L input
- [ ] Efectos visuales de slow-mo

### Phase 4: Polish
- [ ] Partículas elementales
- [ ] Screen shake
- [ ] Audio reactivo
- [ ] Pantallas de carga
- [ ] Transiciones de color

### Phase 5: Contenido
- [ ] Progresión tutorial
- [ ] 3 Boss elementales
- [ ] Modo Arcade infinito

---

## Métricas de Éxito (KPIs)

| Métrica | Target | Medición |
|---------|--------|-----------|
| **Retention D1** | >30% | Usuarios que vuelven al día siguiente |
| **Sessions/Day** | 2.5 | Promedio de sesiones por usuario |
| **Session Length** | 8 min | Duración media de sesión |
| **Chrono Uses** | 5/game | Frecuencia de uso del poder especial |

---

## Deploy

### Mobile
- **iOS:** Apple App Store ($99/año)
- **Android:** Google Play ($25 one-time)

### Nintendo Switch
- **Platform:** Nintendo eShop
- **Engine:** Unity + Platform Extensions
- **Certification:** ~2 semanas review

---

## Próximos Pasos

1. Crear repositorio GitHub
2. Configurar proyecto Unity
3. Implementar BallController básico
4. Iterar elementos
5. Playtest

---

*Documento generado automáticamente*
