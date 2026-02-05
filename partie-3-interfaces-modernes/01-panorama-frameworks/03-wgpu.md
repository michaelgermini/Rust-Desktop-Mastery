# 8.3 wgpu : Rendu GPU Direct

## Présentation

wgpu est une abstraction bas niveau pour le rendu GPU, utilisable pour des UI custom ou des visualisations avancées.

```toml
# Cargo.toml
[dependencies]
wgpu = "0.19"
winit = "0.29"
```

## Quand utiliser wgpu directement

- **Visualisations 3D ou 2D complexes** : Rendu de millions de points
- **Éditeurs graphiques** : Comme Figma, avec rendu GPU custom
- **Jeux ou simulations** : Contrôle total du pipeline graphique
- **Visualisations scientifiques** : Grandes quantités de données

## Architecture typique

```rust
// wgpu est souvent utilisé AVEC egui
use egui_wgpu::Renderer;

// egui utilise wgpu en backend pour le rendu GPU
// Vous pouvez aussi combiner du rendu custom wgpu avec egui
```

### Exemple : Rendu custom avec egui

```rust
use egui::*;
use egui_wgpu::*;

pub struct CustomRenderer {
    // Votre rendu wgpu custom
    custom_pipeline: wgpu::RenderPipeline,
}

impl CustomRenderer {
    pub fn render_custom_content(
        &mut self,
        encoder: &mut wgpu::CommandEncoder,
        view: &wgpu::TextureView,
        egui_output: &egui::FullOutput,
    ) {
        // Rendu custom wgpu
        {
            let mut render_pass = encoder.begin_render_pass(&wgpu::RenderPassDescriptor {
                label: Some("Custom render"),
                color_attachments: &[Some(wgpu::RenderPassColorAttachment {
                    view,
                    resolve_target: None,
                    ops: wgpu::Operations {
                        load: wgpu::LoadOp::Load,
                        store: wgpu::StoreOp::Store,
                    },
                })],
                depth_stencil_attachment: None,
            });
            
            render_pass.set_pipeline(&self.custom_pipeline);
            // Votre rendu custom...
        }
        
        // Puis egui par-dessus
        // egui_wgpu::renderer::render(...)
    }
}
```

## Forces et faiblesses

| Forces | Faiblesses |
|--------|------------|
| Performance maximale | Courbe d'apprentissage raide |
| Contrôle total | Beaucoup de code pour des UI simples |
| Cross-platform (Vulkan, Metal, DX12, WebGPU) | Pas de widgets prêts à l'emploi |

## Cas d'usage spécifiques

### Visualisation de données massive

```rust
// Rendre 1 million de points en temps réel
pub fn render_point_cloud(
    points: &[Point3D],
    camera: &Camera,
    device: &wgpu::Device,
    queue: &wgpu::Queue,
) {
    // wgpu permet de rendre efficacement
    // des quantités massives de données
}
```

### Éditeur graphique

```rust
// Contrôle total du rendu pour un éditeur type Figma
pub struct VectorEditor {
    // Pipeline de rendu vectoriel custom
    vector_pipeline: wgpu::RenderPipeline,
}
```

## Résumé

wgpu est pour les cas où vous avez besoin :
- De performance GPU maximale
- De contrôle total du rendu
- De visualisations complexes
- De combiner rendu custom + UI (via egui_wgpu)

**Note** : Pour la plupart des applications desktop, egui (qui utilise wgpu en backend) est suffisant.
