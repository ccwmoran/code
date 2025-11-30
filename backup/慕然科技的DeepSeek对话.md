[`慕然科技官网`](https://ccwmoran.github.io)
```
(function(Scratch) {
    'use strict';
    
    // DeepSeek对话扩展类
    class DeepSeekDialogs {
        constructor() {
            // 扩展内部状态变量
            this.apiKey = '';
            this.currentModel = 'deepseek-chat';
            this.webSearchEnabled = false;
            this.deepThinkingEnabled = false;
            this.lastResponse = '';
            this.lastDeepThinkingResult = '';
            this.lastThinkingTime = 0;
            this.isProcessing = false;
            
            // 模型映射
            this.modelMap = {
                'V1': 'deepseek-chat',
                'V2': 'deepseek-coder',
                'V3': 'deepseek-reasoner',
                'V3.1': 'deepseek-reasoner'
            };
            
            // DeepSeek品牌颜色
            this.colors = {
                primary: '#1752EF',
                secondary: '#0F3AC4',
                accent: '#4A7AFF'
            };
        }
        
        getInfo() {
            return {
                id: 'kjcxds',
                name: '慕然科技的新DeepSeek对话',
                color1: this.colors.primary,
                color2: this.colors.secondary,
                color3: this.colors.secondary,
                docsURI: 'https://api-docs.deepseek.com/',
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
                        opcode: 'setWebSearch',
                        blockType: Scratch.BlockType.COMMAND,
                        text: '联网搜索 [ENABLED]',
                        arguments: {
                            ENABLED: {
                                type: Scratch.ArgumentType.STRING,
                                menu: 'switch'
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
                        opcode: 'getDeepThinkingResult',
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
                        items: ['V1', 'V2', 'V3', 'V3.1']
                    },
                    switch: {
                        acceptReporters: false,
                        items: ['打开', '关闭']
                    }
                }
            };
        }
        
        // 设置API密钥
        setApiKey(args) {
            const key = args.KEY.trim();
            if (key) {
                this.apiKey = key;
                console.log('API密钥已设置');
            }
        }
        
        // 设置模型
        setModel(args) {
            if (!this.deepThinkingEnabled) {
                const modelKey = args.MODEL;
                this.currentModel = this.modelMap[modelKey] || this.modelMap['V1'];
                console.log(`模型已设置为: ${modelKey} (${this.currentModel})`);
            }
        }
        
        // 设置联网搜索
        setWebSearch(args) {
            this.webSearchEnabled = (args.ENABLED === '打开');
            console.log(`联网搜索: ${this.webSearchEnabled ? '开启' : '关闭'}`);
        }
        
        // 设置深度思考
        setDeepThinking(args) {
            this.deepThinkingEnabled = (args.ENABLED === '打开');
            if (this.deepThinkingEnabled) {
                this.currentModel = this.modelMap['V3.1'];
            }
            console.log(`深度思考: ${this.deepThinkingEnabled ? '开启' : '关闭'}`);
        }
        
        // 获取深度思考结果
        getDeepThinkingResult() {
            return this.lastDeepThinkingResult || '暂无深度思考结果';
        }
        
        // 获取思考用时
        getThinkingTime() {
            if (this.lastThinkingTime === 0) return '0';
            return (Math.round(this.lastThinkingTime / 100) / 10).toString();
        }
        
        // 获取输出结果
        getLastOutput() {
            return this.lastResponse || '暂无输出结果';
        }
        
        // 重置对话
        reset() {
            this.lastResponse = '';
            this.lastDeepThinkingResult = '';
            this.lastThinkingTime = 0;
            this.isProcessing = false;
            console.log('对话已重置');
        }
        
        // 发送消息到DeepSeek API
        sendMessage(args) {
            // 如果正在处理中，返回错误
            if (this.isProcessing) {
                return Promise.resolve('错误：上一个请求正在处理中，请稍后再试');
            }
            
            // 检查API密钥
            if (!this.apiKey) {
                return Promise.resolve('错误：请先设置API密钥');
            }
            
            const message = args.MESSAGE.trim();
            if (!message) {
                return Promise.resolve('错误：消息内容不能为空');
            }
            
            this.isProcessing = true;
            
            // 确定实际使用的模型
            const actualModel = this.deepThinkingEnabled ? this.modelMap['V3.1'] : this.currentModel;
            
            // 构建请求数据
            const requestData = {
                model: actualModel,
                messages: [
                    {
                        role: "user",
                        content: message
                    }
                ],
                stream: false
            };
            
            // 添加联网搜索参数
            if (this.webSearchEnabled) {
                requestData.web_search = true;
            }
            
            // 记录开始时间
            const startTime = Date.now();
            
            // 发送API请求
            return fetch('https://api.deepseek.com/chat/completions', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + this.apiKey
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
                    this.lastResponse = data.choices[0].message.content;
                    
                    // 提取深度思考结果
                    if (data.choices[0].message.reasoning_content) {
                        this.lastDeepThinkingResult = data.choices[0].message.reasoning_content;
                    } else if (this.deepThinkingEnabled) {
                        this.lastDeepThinkingResult = '深度思考内容未返回';
                    } else {
                        this.lastDeepThinkingResult = '深度思考功能未开启';
                    }
                    
                    return this.lastResponse;
                } else {
                    throw new Error('API返回数据异常');
                }
            })
            .catch(error => {
                console.error('DeepSeek API错误:', error);
                this.lastResponse = '';
                this.lastDeepThinkingResult = '';
                return `错误：${error.message}`;
            })
            .finally(() => {
                this.isProcessing = false;
            });
        }
    }
    
    // 注册扩展
    Scratch.extensions.register(new DeepSeekDialogs());
})(Scratch);
```