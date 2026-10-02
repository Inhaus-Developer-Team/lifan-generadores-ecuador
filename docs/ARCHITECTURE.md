# Arquitectura Técnica del Sistema

## LIFAN Power Ecuador — Plataforma Web

---

## 1. Topología del Sistema

El proyecto está diseñado bajo el principio de **Static Web Architecture (JAMstack sin build step)**, lo que garantiza máxima resiliencia, costos de operación cero, y disponibilidad ininterrumpida aun frente a picos masivos de tráfico.

```mermaid
graph LR
    subgraph Clientes
        C1[Navegador Móvil / Desktop]
        C2[Agente AI / LLM Bot]
        C3[Crawler de Google / Meta]
    end

    subgraph GitHub CDN Edge
        GH[GitHub Pages CDN]
        WAF[DDoS Protection & SSL / HTTPS]
    end

    subgraph Archivos Servidos
        INDEX[index.html / index.modern.html]
        ASSETS[Media PNG / WebP / SVG]
        DATA[Schema.org JSON-LD / llms.txt]
    end

    subgraph Canales de Conversión
        WA[API WhatsApp Click-to-Chat]
        TEL[Red Telefónica Móvil Ecuador]
    end

    C1 --> WAF --> GH
    C2 --> WAF --> GH
    C3 --> WAF --> GH

    GH --> INDEX
    GH --> ASSETS
    GH --> DATA

    C1 -.->|Clic en Cotizar| WA
    C1 -.->|Clic en Llamar| TEL
```

---

## 2. Decisiones de Arquitectura Frontend

### 2.1 Por qué HTML Nativo + Tailwind CSS
- **Cero tiempo de compilación (Zero Build Overhead)**: No requiere Node.js, Webpack, Vite ni frameworks con hidratación pesada (React, Next.js, Vue).
- **Time to First Byte (TTFB)** y **First Contentful Paint (FCP)** instantáneos servidos desde los puntos de presencia (PoPs) globales de GitHub Pages.
- **Resistencia al fallo**: No hay dependencias de bases de datos que puedan caer durante una crisis de demanda energética.

### 2.2 Estrategia de Priorización de Recursos (Resource Prioritization)

```mermaid
sequenceDiagram
    participant B as Navegador
    participant S as Servidor GitHub Pages
    participant G as Google Fonts

    B->>S: GET /index.modern.html
    S-->>B: Retorna Documento HTML (200 OK)
    
    par Conexión anticipada
        B->>G: Preconnect a fonts.googleapis.com
        B->>G: Preconnect a fonts.gstatic.com
    and Carga crítica (LCP)
        B->>S: GET Hero Product Image (fetchpriority="high")
    end

    Note over B: Renderizado inicial sin Layout Shift (CLS = 0)

    opt Carga diferida (Offscreen)
        B->>S: GET Imágenes del Catálogo (loading="lazy", decoding="async")
    end
```

---

## 3. Arquitectura de Datos Semánticos (AI & Machine Agents)

Para que los agentes de IA (ChatGPT Search, Perplexity, Claude, Google Gemini) y crawlers obtengan la información del inventario de forma inmediata y sin ambigüedades, la página implementa dos capas de datos:

1. **Schema.org JSON-LD Graph**:
   - `Organization`: Razón social, país (`EC`), ciudad (`Cuenca`), número telefónico de soporte y URL oficial.
   - `ItemList`: Relación formal de los cuatro modelos de generadores con sus campos `name`, `image`, `description`, `sku`, `offers.price` (en USD) y `offers.availability`.
   - `FAQPage`: Banco de preguntas y respuestas técnicas para enriquecer los resultados de búsqueda con fragmentos destacados (Rich Snippets).

2. **Estándar LLMs.txt (`/llms.txt` y `/llms-full.txt`)**:
   - Archivos de texto plano en la raíz del dominio para que cualquier scraper de IA procese el catálogo con 0% de alucinación en capacidades y precios.

---

## 4. Pipeline de CI/CD (GitHub Actions)

El flujo de despliegue automatizado está definido en `.github/workflows/deploy.yml`:

```mermaid
stateDiagram-v2
    [*] --> CommitPush: Push a rama 'main'
    CommitPush --> Checkout: actions/checkout@v4
    Checkout --> Configure: actions/configure-pages@v5
    Configure --> Upload: actions/upload-pages-artifact@v3
    Upload --> Deploy: actions/deploy-pages@v4
    Deploy --> Live: Publicado en GitHub Pages
    Live --> [*]
```
