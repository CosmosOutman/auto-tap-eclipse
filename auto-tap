// ==UserScript==
// @name         Auto Tap Eclipse
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  自动点击 tap.eclipse.xyz 页面中间并定时刷新
// @author       Your name
// @match        *://tap.eclipse.xyz/*
// @match        *://turbotap.xyz/*
// @match        *://*.turbotap.xyz/*
// @grant        GM_setValue
// @grant        GM_getValue
// @run-at       document-start
// ==/UserScript==

(function() {
    'use strict';

    function addStyle(styleString) {
        const style = document.createElement('style');
        style.textContent = styleString;
        document.head.appendChild(style);
    }

    const createUI = () => {
        // 添加样式
        addStyle(`
            .auto-tap-panel {
                position: fixed;
                top: 10px;
                right: 10px;
                background: rgba(0, 0, 0, 0.8);
                padding: 15px;
                border-radius: 8px;
                z-index: 999999;
                color: white;
                font-family: system-ui, -apple-system, sans-serif;
                width: 240px;
            }
            .auto-tap-button {
                width: 100%;
                padding: 8px;
                background: #007AFF;
                border: none;
                border-radius: 4px;
                color: white;
                cursor: pointer;
                margin-top: 10px;
                font-size: 14px;
            }
            .auto-tap-input {
                width: 45px;
                margin: 5px 0;
                padding: 4px;
                border: 1px solid #666;
                border-radius: 4px;
                background: #fff;
                color: #000;
                text-align: center;
            }
            .auto-tap-label {
                display: flex;
                align-items: center;
                margin: 8px 0;
                font-size: 14px;
                gap: 5px;
            }
            .auto-tap-count {
                font-size: 14px;
                margin: 8px 0;
            }
        `);

        // 创建UI元素
        const panel = document.createElement('div');
        panel.className = 'auto-tap-panel';
        panel.innerHTML = `
            <div class="auto-tap-label">
                点击间隔(秒):
                <input type="number" id="minInterval" class="auto-tap-input" value="1" min="0.1" step="0.1">
                ~
                <input type="number" id="maxInterval" class="auto-tap-input" value="5" min="0.1" step="0.1">
            </div>
            <div class="auto-tap-label">
                刷新间隔(h):<input type="number" id="refreshInterval" class="auto-tap-input" value="2" min="0.1" step="0.1">
            </div>
            <div class="auto-tap-count">
                点击次数: <span id="tapCount">0</span>
            </div>
            <button id="tapButton" class="auto-tap-button">开始自动点击</button>
        `;
        document.body.appendChild(panel);

        // 从存储中获取保存的值
        let isRunning = GM_getValue('isRunning', false);
        let clickCount = GM_getValue('clickCount', 0);
        const savedMinInterval = GM_getValue('minInterval', 1);
        const savedMaxInterval = GM_getValue('maxInterval', 5);
        const savedRefreshInterval = GM_getValue('refreshInterval', 2);

        // 设置保存的值到输入框
        document.getElementById('minInterval').value = savedMinInterval;
        document.getElementById('maxInterval').value = savedMaxInterval;
        document.getElementById('refreshInterval').value = savedRefreshInterval;
        document.getElementById('tapCount').textContent = clickCount;
        document.getElementById('tapButton').textContent = isRunning ? '停止自动点击' : '开始自动点击';

        let clickTimer = null;
        let refreshTimer = null;

        // 保存设置
        const saveSettings = () => {
            const minInterval = parseFloat(document.getElementById('minInterval').value) || 1;
            const maxInterval = parseFloat(document.getElementById('maxInterval').value) || 5;
            const refreshHours = parseFloat(document.getElementById('refreshInterval').value) || 2;
            
            GM_setValue('minInterval', minInterval);
            GM_setValue('maxInterval', maxInterval);
            GM_setValue('refreshInterval', refreshHours);
            GM_setValue('isRunning', isRunning);
            GM_setValue('clickCount', clickCount);
            GM_setValue('lastRunTime', Date.now());
        };

        const simulateClick = () => {
            const canvas = document.querySelector('canvas');
            if (canvas) {
                const rect = canvas.getBoundingClientRect();
                // 基准点位置
                const baseX = 200;
                const baseY = 200;

                // 在100x100的范围内随机生成偏移
                const offsetX = (Math.random() - 0.5) * 100;
                const offsetY = (Math.random() - 0.5) * 100;

                // 计算最终的点击位置
                const moveX = baseX + offsetX;
                const moveY = baseY + offsetY;

                // 移动鼠标
                const moveEvent = new MouseEvent('mousemove', {
                    bubbles: true,
                    cancelable: true,
                    clientX: rect.left + moveX,
                    clientY: rect.top + moveY
                });

                canvas.dispatchEvent(moveEvent);

                // 点击位置添加微小随机偏移（让每次点击更自然）
                const clickX = moveX + (Math.random() - 0.5) * 5;
                const clickY = moveY + (Math.random() - 0.5) * 5;

                // 模拟鼠标事件序列
                ['mousedown', 'click', 'mouseup'].forEach(eventType => {
                    const event = new MouseEvent(eventType, {
                        bubbles: true,
                        cancelable: true,
                        clientX: rect.left + clickX,
                        clientY: rect.top + clickY
                    });
                    canvas.dispatchEvent(event);
                });

                // 更新计数
                clickCount++;
                document.getElementById('tapCount').textContent = clickCount;
                GM_setValue('clickCount', clickCount);
            }
        };

        // 输入框值变化时保存设置
        ['minInterval', 'maxInterval', 'refreshInterval'].forEach(id => {
            document.getElementById(id).addEventListener('change', saveSettings);
        });

        document.getElementById('tapButton').onclick = () => {
            isRunning = !isRunning;
            const button = document.getElementById('tapButton');
            button.textContent = isRunning ? '停止自动点击' : '开始自动点击';
            
            // 立即保存状态
            saveSettings();

            if (isRunning) {
                startClicking();
            } else {
                stopClicking();
            }
        };

        // 分离点击逻辑
        const startClicking = () => {
            const minInterval = parseFloat(document.getElementById('minInterval').value) || 1;
            const maxInterval = parseFloat(document.getElementById('maxInterval').value) || 5;
            const refreshHours = parseFloat(document.getElementById('refreshInterval').value) || 2;

            const doClick = () => {
                if (!isRunning) return;
                simulateClick();
                // 生成随机延迟（秒转换为毫秒）
                const randomDelay = (minInterval + Math.random() * (maxInterval - minInterval)) * 1000;
                clickTimer = setTimeout(doClick, randomDelay);
            };
            doClick();

            refreshTimer = setTimeout(() => {
                saveSettings();
                window.location.reload();
            }, refreshHours * 3600 * 1000);
        };

        const stopClicking = () => {
            clearTimeout(clickTimer);
            clearTimeout(refreshTimer);
            isRunning = false;
            GM_setValue('isRunning', false);
        };

        // 自动启动检查移到这里
        const autoStart = () => {
            if (GM_getValue('isRunning', false)) {
                isRunning = true;
                document.getElementById('tapButton').textContent = '停止自动点击';
                startClicking();
            }
        };

        // 延迟执行自动启动
        setTimeout(autoStart, 1000);
    };

    // 将init函数移到外面
    const init = () => {
        if (document.querySelector('canvas')) {
            createUI();
        } else {
            setTimeout(init, 1000);
        }
    };

    // 确保页面加载完成后再运行
    if (document.readyState === 'complete') {
        init();
    } else {
        window.addEventListener('load', init);
    }
})(); 
