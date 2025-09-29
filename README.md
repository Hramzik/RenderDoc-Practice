# Разбор рендеринга в игре Miside с помощью RenderDoc

При выполнении задания был проведён захват и анализ кадра из игры Miside с использованием RenderDoc. В этом отчёте мы подробно разберём наиболее интересные находки, попытаемся увидеть в них изученные на лекциях приёмы и улучшить понимание работы графического пайплайна в играх

### Игровой вид снимка
![Главное меню игры](imgs/result.png)

## Стек вызовов
Всего за снимок происходит 1341 вызов графического API, которые можно разделить на несколько проходов и отдельные вызовы:
| Проход | Время выполнения (мс) |
|-----------------   |----------------------|
| Compute Pass #1    | 0.6 |
| Depth only Pass #1 | 0.9 |
| Depth only Pass #2 | 1.3 |
| Compute Pass #2    | 10.3 |
| Color Pass #1      | 4.2 |
| Color Pass #2      | 7.6 |
| **Общее время кадра** | **36.5** |


Сначала первый Compute Pass копирует какие-то внутренние текстуры
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/internal.png" style="height: 200px; object-fit: contain;">
</div>

Далее выполняется Z-prepass, создающий CameraDepthTexture, которая позже будет использована для оптимизации отрисовки (Depth test)
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/camera-depth.png" style="height: 270px; object-fit: contain;">
    <img src="imgs/depth-test.png" style="height: 270px; object-fit: contain;">
</div>

Второй Depth-only Pass считает shadowmap сцены, который впоследствии будет использован для наложения теней
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/shadowmap-start.png" style="height: 200px; object-fit: contain;">
    <img src="imgs/shadowmap-overdraw.png" style="height: 200px; object-fit: contain;">
    <img src="imgs/shadowmap.png" style="height: 200px; object-fit: contain;">
</div>

Уже на этих этапах можно покрутить используемые меши и при желании выгрузить куда-то себе :)
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/face-poly.png" style="height: 200px; object-fit: contain;">
    <img src="imgs/hair-poly.png" style="height: 200px; object-fit: contain;">
</div>

Далее во втором Compute Pass идет Multi-scale Volumetric Obscurance - Ambient Occlusion на нескольких мипах карты глубин
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/map-low.png" style="height: 200px; object-fit: contain;">
    <img src="imgs/map-high.png" style="height: 200px; object-fit: contain;">
    <img src="imgs/map.png" style="height: 200px; object-fit: contain;">
</div>

Первый Color Pass просто накладывает текстуры
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/tx-cloth.png" style="height: 100px; object-fit: contain;">
    <img src="imgs/tx-hair.png" style="height: 100px; object-fit: contain;">
    <img src="imgs/tx-body.png" style="height: 100px; object-fit: contain;">
    <img src="imgs/tx-face.png" style="height: 100px; object-fit: contain;">
</div>

<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/textured.png" style="height: 300; object-fit: contain;">
</div>

А второй занимается постобработкой - накладывает виньетку и рисует узоры на фоне
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/vignette.png" style="height: 270px; object-fit: contain;">
    <img src="imgs/vignette-result.png" style="height: 270px; object-fit: contain;">
</div>

В качестве proof-of-concept, я отредактировал шейдер виньетки
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/my-shader.png" style="height: 270px; object-fit: contain;">
    <img src="imgs/my-shader-result.png" style="height: 270px; object-fit: contain;">
</div>

В последнюю очередь, как можно было догадаться, рисуется интерфейс, причем на текстуре видны артефакты его создания при отключении альфа-канала
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/interface-1.png" style="height: 200px; object-fit: contain;">
    <img src="imgs/interface-2.png" style="height: 200px; object-fit: contain;">
    <img src="imgs/interfaced.png" style="height: 200px; object-fit: contain;">
</div>

Перед выводом делается blit текстуры, чтобы ее перевернуть (прямо как у меня в 3-ей домашке!)
<div style="display: flex; gap: 10px; align-items: center;">
    <img src="imgs/blit-calls.png" style="height: 200px; object-fit: contain;">
    <img src="imgs/blit-result.png" style="height: 200px; object-fit: contain;">
</div>

Таким образом, на примере этой простой сцены можно продемонстрировать возможности применения RenderDoc и лучше разобраться с тонкостями рендеринга