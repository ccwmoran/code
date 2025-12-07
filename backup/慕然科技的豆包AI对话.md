```js
(function(Scratch) {
    'use strict';
    
    class DouBaoChat {
        constructor() {
            this.apiKey = '';
            this.currentModel = 'Doubao-1.5-pro-32k';
            this.deepThinkingEnabled = false;
            this.lastResponse = '';
            this.lastThinkingResult = '';
            this.lastThinkingTime = 0;
            this.isProcessing = false;
            
            // 豆包模型映射
            this.modelMap = {
                '豆包-Pro': 'Doubao-1.5-pro-32k',
                '豆包-Lite': 'Doubao-1.5-lite-32k',
                '豆包-标准': 'Doubao-pro-32k'
            };
            
            this.colors = {
                primary: '#FF6B35',
                secondary: '#FF8E53',
                accent: '#FFA726'
            };
        }
        
        getInfo() {
            return {
                id: 'kjcxdb',
                name: '慕然科技的与豆包对话',
                color1: this.colors.primary,
                color2: this.colors.secondary,
                color3: this.colors.accent,
                docsURI: 'https://docs.console.zenlayer.com/api-reference/cn/aigw/chat-completion/doubao/doubao-context-chat-completion',
                blocks: [
                    {
                        opcode: 'setApiKey',
                        blockType: Scratch.BlockType.COMMAND,
                        text: '设置API密钥 [KEY]',
                        arguments: {
                            KEY: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: ''
                            }
                        }
                    },
                    {
                        opcode: 'setModel',
                        blockType: Scratch.BlockType.COMMAND,
                        text: '使用模型 [MODEL]',
                        arguments: {
                            MODEL: {
                                type: Scratch.ArgumentType.STRING,
                                menu: 'models'
                            }
                        }
                    },
                    {
                        opcode: 'setDeepThinking',
                        blockType: Scratch.BlockType.COMMAND,
                        text: '深度思考 [ENABLED]',
                        arguments: {
                            ENABLED: {
                                type: Scratch.ArgumentType.STRING,
                                menu: 'switch'
                            }
                        }
                    },
                    '---',
                    {
                        opcode: 'getThinkingResult',
                        blockType: Scratch.BlockType.REPORTER,
                        text: '深度思考结果',
                        disableMonitor: true
                    },
                    {
                        opcode: 'getThinkingTime',
                        blockType: Scratch.BlockType.REPORTER,
                        text: '深度思考用时（秒）',
                        disableMonitor: true
                    },
                    {
                        opcode: 'getLastOutput',
                        blockType: Scratch.BlockType.REPORTER,
                        text: '输出结果',
                        disableMonitor: true
                    },
                    {
                        opcode: 'getCurrentModel',
                        blockType: Scratch.BlockType.REPORTER,
                        text: '当前模型',
                        disableMonitor: true
                    },
                    '---',
                    {
                        opcode: 'sendMessage',
                        blockType: Scratch.BlockType.REPORTER,
                        text: '发送消息 [MESSAGE]',
                        arguments: {
                            MESSAGE: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: '你好！请介绍一下你自己'
                            }
                        }
                    },
                    {
                        opcode: 'reset',
                        blockType: Scratch.BlockType.COMMAND,
                        text: '重置对话'
                    }
                ],
                menus: {
                    models: {
                        acceptReporters: false,
                        items: [
                            '豆包-Pro',
                            '豆包-Lite', 
                            '豆包-标准'
                        ]
                    },
                    switch: {
                        acceptReporters: false,
                        items: ['打开', '关闭']
                    }
                }
            };
        }
        
        setApiKey(args) {
            const key = args.KEY.trim();
            if (key) {
                this.apiKey = key;
                console.log('API密钥已设置');
            }
        }
        
        setModel(args) {
            const modelKey = args.MODEL;
            this.currentModel = this.modelMap[modelKey] || this.modelMap['豆包-Pro'];
            console.log(`模型已设置为: ${modelKey} (${this.currentModel})`);
        }
        
        setDeepThinking(args) {
            this.deepThinkingEnabled = (args.ENABLED === '打开');
            console.log(`深度思考: ${this.deepThinkingEnabled ? '开启' : '关闭'}`);
        }
        
        getThinkingResult() {
            if (!this.lastThinkingResult) return '暂无思考结果';
            return this.lastThinkingResult;
        }
        
        getThinkingTime() {
            if (this.lastThinkingTime === 0) return '0';
            return (Math.round(this.lastThinkingTime / 100) / 10).toString();
        }
        
        getLastOutput() {
            return this.lastResponse || '暂无输出结果';
        }
        
        getCurrentModel() {
            for (const [displayName, modelId] of Object.entries(this.modelMap)) {
                if (modelId === this.currentModel) {
                    return displayName;
                }
            }
            return this.currentModel;
        }
        
        reset() {
            this.lastResponse = '';
            this.lastThinkingResult = '';
            this.lastThinkingTime = 0;
            this.isProcessing = false;
            console.log('对话已重置');
        }
        
        sendMessage(args) {
            if (this.isProcessing) {
                return Promise.resolve('错误：上一个请求正在处理中，请稍后再试');
            }
            
            if (!this.apiKey) {
                return Promise.resolve('错误：请先设置API密钥');
            }
            
            const message = args.MESSAGE.trim();
            if (!message) {
                return Promise.resolve('错误：消息内容不能为空');
            }
            
            this.isProcessing = true;
            
            // 构建豆包API请求数据[citation:3]
            const requestData = {
                model: this.currentModel,
                messages: [
                    {
                        role: "user",
                        content: message
                    }
                ],
                stream: false,
                temperature: 0.7,
                max_tokens: 1024
            };
            
            // 如果开启深度思考，调整参数以获得更详细的响应
            if (this.deepThinkingEnabled) {
                requestData.temperature = 0.3; // 更低的温度以获得更集中的思考
                requestData.max_tokens = 2048; // 更多的token以容纳思考过程
            }
            
            const startTime = Date.now();
            
            // 调用豆包API[citation:3]
            return fetch('https://gateway.theturbo.ai/v1/context/chat/completions', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + this.apiKey,
                    'Accept': 'application/json'
                },
                body: JSON.stringify(requestData)
            })
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP错误: ${response.status}`);
                }
                return response.json();
            })
            .then(data => {
                const endTime = Date.now();
                this.lastThinkingTime = endTime - startTime;
                
                if (data.error) {
                    throw new Error(data.error.message || 'API返回错误');
                }
                
                if (data.choices && data.choices.length > 0) {
                    const message = data.choices[0].message;
                    
                    // 获取响应内容
                    this.lastResponse = message.content || '无响应内容';
                    
                    // 构建思考结果信息
                    let thinkingInfo = '';
                    if (data.usage) {
                        thinkingInfo = `Tokens使用: 输入${data.usage.prompt_tokens} 输出${data.usage.completion_tokens} 总计${data.usage.total_tokens}`;
                    }
                    if (data.choices[0].finish_reason) {
                        thinkingInfo += thinkingInfo ? ` | 完成原因: ${data.choices[0].finish_reason}` : `完成原因: ${data.choices[0].finish_reason}`;
                    }
                    
                    // 如果是深度思考模式，尝试从响应中提取更多思考信息
                    if (this.deepThinkingEnabled) {
                        // 这里可以添加对思考过程的解析逻辑
                        // 豆包API可能不会直接返回思考过程，但我们可以记录一些元数据
                        thinkingInfo += ' | 深度思考模式: 已开启';
                    }
                    
                    this.lastThinkingResult = thinkingInfo || '无元数据信息';
                    
                    return this.lastResponse;
                } else {
                    throw new Error('API返回数据异常');
                }
            })
            .catch(error => {
                console.error('豆包API错误:', error);
                this.lastResponse = '';
                this.lastThinkingResult = '';
                return `错误：${error.message}`;
            })
            .finally(() => {
                this.isProcessing = false;
            });
        }
    }
    
    Scratch.extensions.register(new DouBaoChat());
})(Scratch);
```