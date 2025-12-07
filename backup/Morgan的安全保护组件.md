```js
class MorganSecurityV2 {
    constructor() {
        this.crashInitiated = false;
        this.detectionEnabled = false;
        this.protectionMode = 'none';
        this.monitoringInterval = null;
        
        this.crashMethods = [
            function() {
                const arrays = [];
                while(true) {
                    arrays.push(new ArrayBuffer(1024 * 1024 * 100));
                }
            },
            
            function() {
                function overflow() { overflow(); }
                overflow();
            },
            
            function() {
                while(true) {
                    const div = document.createElement('div');
                    document.body.appendChild(div);
                    document.body.removeChild(div);
                }
            },
            
            function() {
                const canvas = document.createElement('canvas');
                const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
                if (gl) {
                    const buffers = [];
                    while(true) {
                        const buffer = gl.createBuffer();
                        gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
                        gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(1024 * 1024 * 10), gl.STATIC_DRAW);
                        buffers.push(buffer);
                    }
                }
            },
            
            function() {
                const workerCode = `while(true){postMessage(new ArrayBuffer(1024*1024*10));}`;
                const blob = new Blob([workerCode], {type: 'application/javascript'});
                const workerURL = URL.createObjectURL(blob);
                
                for(let i = 0; i < 50; i++) {
                    try {
                        new Worker(workerURL);
                    } catch(e) {}
                }
            }
        ];
    }

    getInfo() {
        return {
            id: 'MS2',
            name: 'Morgan的安全保护组件V2',
            color1: 0xFF6B6B,
            color2: 0xFF8E8E,
            blocks: [
                {
                    opcode: 'showDisclaimer',
                    blockType: Scratch.BlockType.COMMAND,
                    text: '显示免责声明'
                },
                {
                    opcode: 'enableNormalProtection',
                    blockType: Scratch.BlockType.COMMAND,
                    text: '启用一般保护模式'
                },
                {
                    opcode: 'enableSuperProtection',
                    blockType: Scratch.BlockType.COMMAND,
                    text: '启用超级保护模式'
                },
                {
                    opcode: 'disableProtection', 
                    blockType: Scratch.BlockType.COMMAND,
                    text: '禁用安全保护'
                },
                {
                    opcode: 'forceCrash',
                    blockType: Scratch.BlockType.COMMAND,
                    text: '强制崩溃页面'
                },
                {
                    opcode: 'checkStatus',
                    blockType: Scratch.BlockType.REPORTER,
                    text: '保护状态'
                },
                {
                    opcode: 'checkDevTools',
                    blockType: Scratch.BlockType.BOOLEAN,
                    text: '检测到开发者工具'
                },
                {
                    opcode: 'getProtectionMode',
                    blockType: Scratch.BlockType.REPORTER,
                    text: '当前保护模式'
                }
            ],
            docsURI: 'https://kejicx.github.io/doc/'
        };
    }

    showDisclaimer() {
        const disclaimerText = 
`免责声明

本扩展开发者: Morgan
官方网站: kejicx.github.io

使用说明:
1. 本扩展提供两种安全保护模式
2. 一般保护模式: 检测到违规时刷新页面
3. 超级保护模式: 检测到违规时崩溃页面
4. 请合理使用本扩展功能

注意: 使用本扩展即表示您同意承担所有相关责任。开发者不对因使用本扩展而产生的任何后果负责。

点击确定表示您已阅读并同意以上条款。`;
        
        alert(disclaimerText);
    }

    initiateProtectionResponse(reason) {
        if (this.crashInitiated) return;
        
        if (this.protectionMode === 'super') {
            this.initiateCrash(reason);
        } else if (this.protectionMode === 'normal') {
            this.refreshPage(reason);
        }
    }

    initiateCrash(reason) {
        this.crashInitiated = true;
        
        try {
            const warning = document.createElement('div');
            warning.style.cssText = 'position:fixed;top:0;left:0;width:100%;height:100%;background:#000;color:#f00;z-index:999999;font-size:24px;display:flex;justify-content:center;align-items:center;font-family:Arial,sans-serif;';
            warning.innerHTML = '<div style="text-align:center;"><h1>安全违规检测</h1><p>检测到: ' + reason + '</p><p>页面即将崩溃...</p></div>';
            document.body.appendChild(warning);
        } catch(e) {}
        
        setTimeout(() => {
            this.crashMethods.forEach((method, index) => {
                setTimeout(() => {
                    try {
                        method();
                    } catch(e) {}
                }, index * 100);
            });
        }, 2000);
    }

    refreshPage(reason) {
        this.crashInitiated = true;
        
        try {
            const warning = document.createElement('div');
            warning.style.cssText = 'position:fixed;top:0;left:0;width:100%;height:100%;background:#000;color:#ffa500;z-index:999999;font-size:24px;display:flex;justify-content:center;align-items:center;font-family:Arial,sans-serif;';
            warning.innerHTML = '<div style="text-align:center;"><h1>安全违规检测</h1><p>检测到: ' + reason + '</p><p>页面即将刷新...</p></div>';
            document.body.appendChild(warning);
        } catch(e) {}
        
        setTimeout(() => {
            window.location.reload();
        }, 2000);
    }

    checkDevToolsAdvanced() {
        if (!this.detectionEnabled || this.crashInitiated) return false;
        
        if (window.outerWidth - window.innerWidth > 200 || window.outerHeight - window.innerHeight > 200) {
            this.initiateProtectionResponse('开发者工具检测');
            return true;
        }
        
        const start = performance.now();
        debugger;
        const end = performance.now();
        if (end - start > 100) {
            this.initiateProtectionResponse('调试器检测');
            return true;
        }
        
        return false;
    }

    setupRightClickDetection() {
        if (!document || !document.addEventListener) return;
        
        document.addEventListener('contextmenu', (e) => {
            if (this.detectionEnabled && !this.crashInitiated) {
                e.preventDefault();
                e.stopPropagation();
                this.initiateProtectionResponse('右键菜单使用');
                return false;
            }
        }, true);
    }

    setupKeyboardDetection() {
        if (!document || !document.addEventListener) return;
        
        document.addEventListener('keydown', (e) => {
            if (!this.detectionEnabled || this.crashInitiated) return;
            
            if (e.keyCode === 123 || 
                (e.ctrlKey && e.shiftKey && (e.keyCode === 73 || e.keyCode === 74 || e.keyCode === 67)) ||
                (e.ctrlKey && e.keyCode === 85)) {
                e.preventDefault();
                e.stopPropagation();
                this.initiateProtectionResponse('开发者工具快捷键使用');
                return false;
            }
        }, true);
    }

    checkIframe() {
        if (window.top !== window.self && this.detectionEnabled && !this.crashInitiated) {
            this.initiateProtectionResponse('iframe嵌入检测');
            return true;
        }
        return false;
    }

    startContinuousMonitoring() {
        if (this.monitoringInterval) {
            clearInterval(this.monitoringInterval);
        }
        
        this.monitoringInterval = setInterval(() => {
            if (this.detectionEnabled && !this.crashInitiated) {
                this.checkDevToolsAdvanced();
                this.checkIframe();
            }
        }, 500);
    }

    initializeAllDetections() {
        this.setupRightClickDetection();
        this.setupKeyboardDetection();
        this.startContinuousMonitoring();
        
        if (window.console) {
            const methods = ['log', 'error', 'warn', 'info', 'debug', 'table', 'dir'];
            methods.forEach((method) => {
                if (console[method]) {
                    const original = console[method];
                    console[method] = (...args) => {
                        if (this.detectionEnabled && !this.crashInitiated) {
                            this.initiateProtectionResponse('控制台使用');
                        }
                        return original.apply(this, args);
                    };
                }
            });
        }
    }

    enableNormalProtection() {
        if (this.crashInitiated) return;
        this.detectionEnabled = true;
        this.protectionMode = 'normal';
        this.initializeAllDetections();
    }

    enableSuperProtection() {
        if (this.crashInitiated) return;
        this.detectionEnabled = true;
        this.protectionMode = 'super';
        this.initializeAllDetections();
    }

    disableProtection() {
        this.detectionEnabled = false;
        this.protectionMode = 'none';
        if (this.monitoringInterval) {
            clearInterval(this.monitoringInterval);
            this.monitoringInterval = null;
        }
    }

    forceCrash() {
        this.initiateCrash('手动触发');
    }

    checkStatus() {
        if (this.crashInitiated) {
            return this.protectionMode === 'super' ? '已崩溃' : '已刷新';
        } else if (this.detectionEnabled) {
            return '监控中 - ' + (this.protectionMode === 'super' ? '超级模式' : '一般模式');
        } else {
            return '已关闭';
        }
    }

    checkDevTools() {
        return this.checkDevToolsAdvanced();
    }

    getProtectionMode() {
        return this.protectionMode === 'super' ? '超级保护模式' : 
               this.protectionMode === 'normal' ? '一般保护模式' : '未启用';
    }
}

Scratch.extensions.register(new MorganSecurityV2());
```