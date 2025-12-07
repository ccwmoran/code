```js
(function(Scratch) {
    'use strict';
    
    class ChatGPTExtension {
        constructor() {
            this.apiKey = '';
            this.currentModel = 'gpt-3.5-turbo';
            this.webSearchEnabled = false;
            this.deepThinkingEnabled = false;
            this.lastResponse = '';
            this.lastThinkingResult = '';
            this.lastThinkingTime = 0;
            this.conversationHistory = [];
            this.isProcessing = false;
            
            // OpenAI模型映射
            this.modelMap = {
                'GPT-3.5 Turbo': 'gpt-3.5-turbo',
                'GPT-4': 'gpt-4',
                'GPT-4 Turbo': 'gpt-4-turbo-preview'
            };
            
            this.colors = {
                primary: '#10A37F',
                secondary: '#0D8E6F',
                accent: '#0B7A5F'
            };
        }
        
        getInfo() {
            return {
                id: 'kjcxchatgpt',
                name: '慕然科技的ChatGPT对话',
                color1: this.colors.primary,
                color2: this.colors.secondary,
                color3: this.colors.accent,
                docsURI: 'https://platform.openai.com/docs/api-reference/chat',
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
                        opcode: 'resetConversation',
                        blockType: Scratch.BlockType.COMMAND,
                        text: '重置对话历史'
                    }
                ],
                menus: {
                    models: {
                        acceptReporters: false,
                        items: [
                            'GPT-3.5 Turbo',
                            'GPT-4',
                            'GPT-4 Turbo'
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
            this.currentModel = this.modelMap[modelKey] || this.modelMap['GPT-3.5 Turbo'];
            console.log(`模型已设置为: ${modelKey} (${this.currentModel})`);
        }
        
        setWebSearch(args) {
            this.webSearchEnabled = (args.ENABLED === '打开');
            console.log(`联网搜索: ${this.webSearchEnabled ? '开启' : '关闭'}`);
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
        
        resetConversation() {
            this.conversationHistory = [];
            this.lastResponse = '';
            this.lastThinkingResult = '';
            this.lastThinkingTime = 0;
            console.log('对话历史已重置');
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
            
            // 构建对话消息[citation:1]
            const messages = [...this.conversationHistory];
            
            // 添加系统提示词
            let systemMessage = '你是一个有帮助的AI助手。';
            if (this.deepThinkingEnabled) {
                systemMessage += '请在进行深入思考和分析后给出回答。';
            }
            if (this.webSearchEnabled) {
                systemMessage += '当前时间：' + new Date().toLocaleString() + '。请基于最新信息回答。';
            }
            
            messages.unshift({ role: 'system', content: systemMessage });
            messages.push({ role: 'user', content: message });
            
            // 构建OpenAI API请求数据[citation:1]
            const requestData = {
                model: this.currentModel,
                messages: messages,
                stream: false,
                temperature: this.deepThinkingEnabled ? 0.7 : 0.9,
                max_tokens: this.deepThinkingEnabled ? 2000 : 1000
            };
            
            const startTime = Date.now();
            
            // 调用OpenAI API[citation:1]
            return fetch('https://api.openai.com/v1/chat/completions', {
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
                    const message = data.choices[0].message;
                    
                    // 保存到对话历史
                    this.conversationHistory.push({ role: 'user', content: message });
                    this.conversationHistory.push(message);
                    
                    // 限制对话历史长度
                    if (this.conversationHistory.length > 20) {
                        this.conversationHistory = this.conversationHistory.slice(-20);
                    }
                    
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
                    
                    // 如果是深度思考模式，记录额外的思考信息
                    if (this.deepThinkingEnabled) {
                        thinkingInfo += ' | 深度思考模式: 已开启';
                    }
                    
                    this.lastThinkingResult = thinkingInfo || '无元数据信息';
                    
                    return this.lastResponse;
                } else {
                    throw new Error('API返回数据异常');
                }
            })
            .catch(error => {
                console.error('OpenAI API错误:', error);
                this.lastResponse = '';
                this.lastThinkingResult = '';
                return `错误：${error.message}`;
            })
            .finally(() => {
                this.isProcessing = false;
            });
        }
    }
    
    Scratch.extensions.register(new ChatGPTExtension());
})(Scratch);
```