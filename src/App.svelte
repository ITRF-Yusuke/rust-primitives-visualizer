<script lang="ts">
  // 設定
  let bitSize = 8; // 8, 16, 32, 64
  let isSigned = false;
  
  // BigIntで状態を管理 (2^64まで対応)
  let value = 0n;

  // ビット数やモードが変わった時のリセット処理
  $: maxUnsigned = (1n << BigInt(bitSize));
  $: maxSignedThreshold = (1n << BigInt(bitSize - 1));

  // 各ビットの重みリストを生成
  $: weights = Array.from({ length: bitSize }, (_, i) => {
    let power = BigInt(bitSize - 1 - i);
    if (isSigned && i === 0) {
      return -(1n << power); // 最上位ビットを負にする
    }
    return 1n << power;
  });

  // 10進数表示用計算
  $: displayValue = calculateDisplayValue(value, isSigned, bitSize);

  function calculateDisplayValue(val: bigint, signed: boolean, size: number) {
    if (!signed) return val.toString();
    let threshold = 1n << BigInt(size - 1);
    let total = 1n << BigInt(size);
    return (val >= threshold ? val - total : val).toString();
  }

  function toggleBit(index: number) {
    let power = BigInt(bitSize - 1 - index);
    value = value ^ (1n << power);
  }

  function step(dir: number) {
    let total = 1n << BigInt(bitSize);
    value = (value + BigInt(dir) + total) % total;
  }

  function changeBitSize(size: number) {
    bitSize = size;
    value = 0n; // サイズ変更時は一旦リセット
  }
</script>

<main>
  <div class="panel-container" style="max-width: {bitSize > 16 ? '900px' : 'auto'};">
    <h1 class="panel-title">Rust Integer Visualizer // <span class="mode-indicator">{isSigned ? 'i' : 'u'}{bitSize}</span></h1>

    <div class="controls-row">
      <div class="control-group">
        <div class="switch-label">BIT SIZE</div>
        <div class="switch-wrapper">
          {#each [8, 16, 32, 64] as size}
            <button class:selected={bitSize === size} on:click={() => changeBitSize(size)}>{size}</button>
          {/each}
        </div>
      </div>

      <div class="control-group">
        <div class="switch-label">TYPE</div>
        <div class="switch-wrapper">
          <button class:selected={!isSigned} on:click={() => isSigned = false}>Unsigned</button>
          <button class:selected={isSigned} on:click={() => isSigned = true}>Signed</button>
        </div>
      </div>
    </div>

    <div class="visualizer-interface" class:vertical={bitSize > 8}>
      <div class="bits-grid" style="grid-template-columns: repeat({bitSize > 8 ? 8 : bitSize}, 1fr);">
        {#each weights as weight, i}
          {@const isSet = (value & (1n << BigInt(bitSize - 1 - i))) !== 0n}
          <div class="bit-column">
            <div class="weight-label">{weight.toLocaleString()}</div>
            <div class="led-light {isSet ? 'on' : 'off'}"></div>
            <button class="push-button-round" on:click={() => toggleBit(i)}></button>
          </div>
        {/each}
      </div>

      <div class="decimal-section">
        <div class="seven-segment-display">
          {displayValue}
        </div>
        <div class="adjust-buttons">
          <button class="push-button-rect" on:click={() => step(1)}>+</button>
          <button class="push-button-rect" on:click={() => step(-1)}>−</button>
        </div>
      </div>
    </div>
  </div>
</main>

<style>
  /* 以前のスタイルを継承・拡張 */
  :global(body) {
    margin: 0; background-color: #1a1a1a; display: flex; justify-content: center; align-items: center; min-height: 100vh; font-family: sans-serif;
  }

  .panel-container {
    background: linear-gradient(145deg, #444, #222); padding: 2rem; border-radius: 12px; border: 4px solid #111; color: #ddd; box-shadow: 0 20px 50px rgba(0,0,0,0.7);
  }

  .controls-row {
    display: flex; gap: 2rem; margin-bottom: 2rem; border-bottom: 1px solid #555; padding-bottom: 1.5rem;
  }

  .switch-label { font-size: 0.7rem; color: #888; margin-bottom: 0.5rem; font-weight: bold; }
  
  .switch-wrapper { display: flex; background: #000; border-radius: 4px; padding: 2px; }
  
  .switch-wrapper button {
    background: transparent; border: none; color: #666; padding: 0.4rem 1rem; cursor: pointer; font-weight: bold; transition: all 0.2s;
  }
  
  .switch-wrapper button.selected { background: #444; color: #fff; border-radius: 2px; }

  .visualizer-interface { display: flex; gap: 2rem; align-items: center; }
  .visualizer-interface.vertical { flex-direction: column; }

  /* ビットのグリッド表示 */
  .bits-grid {
    display: grid; gap: 1rem; background: #222; padding: 1.5rem; border-radius: 8px; box-shadow: inset 0 2px 10px rgba(0,0,0,0.5);
  }

  .bit-column { display: flex; flex-direction: column; align-items: center; gap: 0.5rem; }

  .weight-label {
    font-family: 'Courier New', monospace; font-size: 0.6rem; color: #999; height: 1rem; 
    max-width: 60px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
  }

  /* LEDとボタンのスタイルは前回同様 */
  .led-light {
    width: 24px; height: 24px; border-radius: 50%; background: #300; border: 2px solid #111; transition: 0.1s;
  }
  .led-light.on {
    background: #f00; box-shadow: 0 0 15px #f00, inset 0 -2px 5px rgba(255,255,255,0.4);
  }

  .push-button-round {
    width: 24px; height: 24px; border-radius: 50%; border: none; 
    background: linear-gradient(#666, #333); box-shadow: 0 3px 0 #000; cursor: pointer;
  }
  .push-button-round:active { transform: translateY(3px); box-shadow: 0 0 0 #000; }

  /* 7セグメント表示 */
  .decimal-section { display: flex; flex-direction: column; align-items: center; gap: 1rem; }
  .seven-segment-display {
    font-family: 'monospace'; font-size: 3rem; color: #f30; background: #000; 
    padding: 0.5rem 1rem; border-radius: 4px; border: 3px inset #444; 
    min-width: 250px; text-align: right; text-shadow: 0 0 10px #f30;
  }

  .push-button-rect {
    font-size: 1.5rem; padding: 0.5rem 2rem; background: #555; color: white; border: none; border-radius: 4px; box-shadow: 0 4px 0 #222; cursor: pointer;
  }
  .push-button-rect:active { transform: translateY(4px); box-shadow: 0 0 0 #222; }

  .panel-title { font-size: 1.2rem; margin-top: 0; color: #888; }
  .mode-indicator { color: #ff8c00; }
</style>