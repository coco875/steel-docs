---
title: Einen neuen Block hinzufügen (Grundlegende Einrichtung)
description: Grundlegende Anleitung zum Hinzufügen eines neuen Blocks ohne Verhalten mit Hinweisen für Verhaltensimplementierung
---

> ⚠️ Dies ist nur die grundlegende Einrichtung und **bietet noch keine Funktionalität**.

---

## 1. Block-Modul registrieren

Füge dein Block-Modul hinzu zu:

```
steel-core/src/behavior/blocks/mod.rs
```

Es sollte so aussehen:

```rust
mod iron_bars_block;
pub use iron_bars_block::IronBarsBlock;
```

---

## 2. Struct-Namen überprüfen

Überprüfe nochmals, dass dein **Struct-Name korrekt** ist, da er überall übereinstimmen muss, wo er referenziert wird.

---

## 3. Struct zu den generierten Blöcken hinzufügen

Jetzt müssen wir das Struct zur generierten Block-Liste hinzufügen.
Dies geschieht in:

```
steel-core/build/blocks.rs
```

Wenn du verstehen möchtest, was intern passiert, ist die Funktion
`generate_registrations` interessant zu lesen — aber **es ist nicht erforderlich**, um deinen Block zum Laufen zu bringen.

---

## 4. Fokus auf die Build-Funktion

Wir konzentrieren uns nun auf die `build`-Funktion in der geöffneten Datei.

⚠️ **Wichtig:**
Nur **neuen Code hinzufügen**.
**Keinen bestehenden Code entfernen oder ändern**, da dies Blöcke anderer Mitwirkender beschädigen könnte.

---

## 5. Einen veränderbaren Vektor erstellen

Erstelle zunächst einen veränderbaren Vektor mit einem beschreibenden Namen:

```rust
let mut iron_bar_blocks = Vec::new();
```

---

## 6. Match-Statement erweitern

Füge deinen Block-Struct-Namen zum `match`-Statement hinzu.
Nochmals: **nur deine Zeile hinzufügen**, keine anderen entfernen.

```rust
for block in blocks {
    let const_ident = to_const_ident(&block.name);
    match block.class.as_str() {
        ...
        "IronBarsBlock" => iron_bar_blocks.push(const_ident),
        _ => {}
    }
}
```

---

## 7. Block-Typ definieren

Definiere nun den Block-Typ-Identifier:

```rust
let iron_bar_type = Ident::new("IronBarsBlock", Span::call_site());
```

---

## 8. Registrierungen generieren

Als Nächstes die Registrierungen generieren:

```rust
let iron_bar_registrations =
    generate_registrations(iron_bar_blocks.iter(), &iron_bar_type);
```

---

## 9. Registrierungen zur Ausgabe hinzufügen

⚠️ **Sei hier sehr vorsichtig!**

* Das `#` vor dem Registrierungsnamen ist **erforderlich**
* Es verhindert Namenskollisionen mit Rust-Keywords
* Füge **kein** abschließendes Komma hinzu — dieser Code wird in eine andere Datei generiert

Beispiel:

```rust
let output = quote! {
    //! Generated block behavior assignments.

    use steel_registry::vanilla_blocks;
    use crate::behavior::BlockBehaviorRegistry;
    use crate::behavior::blocks::{
        CraftingTableBlock,
        CropBlock,
        EndPortalFrameBlock,
        FarmlandBlock,
        FenceBlock,
        RotatedPillarBlock,
        BarBlock
    };

    pub fn register_block_behaviors(registry: &mut BlockBehaviorRegistry) {
        ...
        #iron_bar_registrations
    }
};
```

---

## 10. Projekt kompilieren

Kompiliere nun das Projekt und lass Rust (und das Build-System) seine Magie wirken.

Nach der Kompilierung sollte dein Block erscheinen in:

```
steel-core/src/behavior/generated/blocks.rs
```

### Fehlerbehebung

Wenn dein Block fehlt:

1. Lösche den `generated`-Ordner
2. Führe aus:

   ```
   cargo clean
   ```
3. Kompiliere erneut

Dies löst das Problem normalerweise.

---

# Verhalten zum Block hinzufügen

An diesem Punkt **macht der Block nichts**.
Um Verhalten hinzuzufügen, musst du die erforderlichen Methoden in `BlockBehaviour` in deiner Block-Datei implementieren (z.B. `iron_bars_block.rs`).

👉 Ein guter Ansatz ist, sich **bestehende Blöcke** mit ähnlichem Verhalten anzusehen und sie als Referenz zu verwenden.

---

## Arbeiten mit Block-States

### Nachbar-Block-State abrufen

```rust
let west_pos = Direction::West.relative(pos);
let west_state = world.get_block_state(&west_pos);
```

Ein `BlockState` enthält **alle Informationen** über diesen spezifischen Block.

---

### Block-State-Eigenschaften ändern

Einen Wert setzen:

```rust
state.set_value(&BlockStateProperties::WEST, true);
```

Einen Wert abrufen funktioniert genauso, nur umgekehrt.

---

## Nachbarblöcke oder Tags überprüfen

Um zu überprüfen, ob ein benachbarter Block zu einer bestimmten Blockgruppe gehört (z.B. Mauern oder Gitterstäbe):

```rust
let walls_tag = Identifier::vanilla_static("walls");
if REGISTRY.blocks.is_in_tag(neighbor_block, &walls_tag) {
    return true;
}
```

---

Das war's — du hast nun die **Grundstruktur** eingerichtet und kannst mit der Implementierung von echtem Verhalten beginnen 🚀

## Weitere nützliche Ressourcen
Derzeit keine verfügbar
