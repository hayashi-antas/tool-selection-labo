```mermaid
graph TD
    %% 全体のスタイル
    classDef default font-family:sans-serif, stroke-width:2px;
    classDef highlight fill:#e1f5fe,stroke:#01579b;
    classDef success fill:#e8f5e9,stroke:#2e7d32;

    %% ステップ1: そのまま
    PDF1[PDF] -->|そのまま| Gemini1((Gemini))
    Gemini1 --> Bad{精度だめ... 😟}

    %% ステップ2: 画像化
    PDF2[PDF] -->|一部を拡大| Crop[切り取って画像化！]
    Crop --> Gemini2((Gemini))
    Gemini2 --> Good{精度100%！ ✨}

    %% ステップ3: 究極の解決策
    PDF3[PDF] --> H[ヘッダーのみ]
    PDF3 --> T1[表 1]
    PDF3 --> T2[表 2]
    
    H & T1 & T2 --> Gemini3((Geminiで統合))
    Gemini3 --> Final[[精度100%の表完成!!]]
    Final --> Win(えっへん! 😤)

    %% スタイル適用
    class PDF1,PDF2,PDF3 highlight;
    class Final,Win success;
```
