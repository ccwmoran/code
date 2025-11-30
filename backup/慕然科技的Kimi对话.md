```
//慕然科技官网ccwmoran.github.io
class MoranKimiExtension {
    constructor() {
        this.apiKey = '';
        this.model = 'moonshot-v1-8k';
        this.webSearch = false;
        this.deepThinking = false;
        this.lastResponse = null;
        this.lastThinkingTime = 0;
        this.isLoading = false;
    }

    getInfo() {
        return {
            id: 'morankimi',
            name: '慕然科技的Kimi对话',
            color: '#FF6B00',
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
                    opcode: 'toggleWebSearch',
                    blockType: Scratch.BlockType.COMMAND,
                    text: '打开/关闭联网搜索 [STATE]',
                    arguments: {
                        STATE: {
                            type: Scratch.ArgumentType.STRING,
                            menu: 'toggleStates'
                        }
                    }
                },
                {
                    opcode: 'toggleDeepThinking',
                    blockType: Scratch.BlockType.COMMAND,
                    text: '打开/关闭深度思考 [STATE]',
                    arguments: {
                        STATE: {
                            type: Scratch.ArgumentType.STRING,
                            menu: 'toggleStates'
                        }
                    }
                },
                {
                    opcode: 'chatWithKimi',
                    blockType: Scratch.BlockType.COMMAND,
                    text: '向Kimi提问 [PROMPT]',
                    arguments: {
                        PROMPT: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: '你好，请介绍一下你自己'
                        }
                    }
                },
                {
                    opcode: 'getResponse',
                    blockType: Scratch.BlockType.REPORTER,
                    text: '输出结果',
                    disableMonitor: true
                },
                {
                    opcode: 'getThinkingTime',
                    blockType: Scratch.BlockType.REPORTER,
                    text: '深度思考用时',
                    disableMonitor: true
                },
                {
                    opcode: 'getDeepThinkingResult',
                    blockType: Scratch.BlockType.REPORTER,
                    text: '深度思考结果',
                    disableMonitor: true
                },
                {
                    opcode: 'isLoading',
                    blockType: Scratch.BlockType.BOOLEAN,
                    text: '正在加载中？',
                    disableMonitor: true
                },
                {
                    opcode: 'clearHistory',
                    blockType: Scratch.BlockType.COMMAND,
                    text: '清除对话历史'
                }
            ],
            menus: {
                models: {
                    items: [
                        {
                            text: 'moonshot-v1-8k',
                            value: 'moonshot-v1-8k'
                        },
                        {
                            text: 'moonshot-v1-32k',
                            value: 'moonshot-v1-32k'
                        },
                        {
                            text: 'moonshot-v1-128k',
                            value: 'moonshot-v1-128k'
                        }
                    ]
                },
                toggleStates: {
                    items: [
                        {
                            text: '打开',
                            value: 'on'
                        },
                        {
                            text: '关闭',
                            value: 'off'
                        }
                    ]
                }
            }
        };
    }

    setApiKey(args) {
        this.apiKey = args.KEY.trim();
        console.log('API密钥已设置');
    }

    setModel(args) {
        this.model = args.MODEL;
        console.log(`模型设置为: ${this.model}`);
    }

    toggleWebSearch(args) {
        this.webSearch = args.STATE === 'on';
        console.log(`联网搜索${this.webSearch ? '打开' : '关闭'}`);
    }

    toggleDeepThinking(args) {
        this.deepThinking = args.STATE === 'on';
        console.log(`深度思考${this.deepThinking ? '打开' : '关闭'}`);
    }

    async chatWithKimi(args) {
        if (!this.apiKey) {
            console.error('请先设置API密钥');
            return;
        }

        const prompt = args.PROMPT;
        
        this.isLoading = true;
        
        try {
            const startTime = Date.now();
            
            const requestBody = {
                model: this.model,
                messages: [
                    {
                        role: 'user',
                        content: prompt
                    }
                ],
                temperature: 0.7,
                max_tokens: 2048
            };

            // 添加联网搜索和深度思考参数
            if (this.webSearch) {
                requestBody.web_search = true;
            }
            
            if (this.deepThinking) {
                requestBody.deep_thinking = true;
            }

            const response = await fetch('https://api.moonshot.cn/v1/chat/completions', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${this.apiKey}`
                },
                body: JSON.stringify(requestBody)
            });

            if (!response.ok) {
                throw new Error(`API请求失败: ${response.status} ${response.statusText}`);
            }

            const data = await response.json();
            const endTime = Date.now();
            
            this.lastResponse = data;
            this.lastThinkingTime = endTime - startTime;
            
            console.log(`Kimi响应接收成功，思考用时: ${this.lastThinkingTime}ms`);
            
        } catch (error) {
            console.error(`调用Kimi API出错: ${error.message}`);
            this.lastResponse = null;
            this.lastThinkingTime = 0;
        } finally {
            this.isLoading = false;
        }
    }

    getResponse() {
        if (this.isLoading) {
            return '正在思考中...';
        }
        
        if (!this.lastResponse || !this.lastResponse.choices || this.lastResponse.choices.length === 0) {
            return '暂无响应';
        }
        
        return this.lastResponse.choices[0].message.content || '无内容';
    }

    getThinkingTime() {
        return this.lastThinkingTime;
    }

    getDeepThinkingResult() {
        if (!this.lastResponse || !this.lastResponse.choices || this.lastResponse.choices.length === 0) {
            return '暂无深度思考结果';
        }
        
        // 如果有深度思考结果，从响应中提取
        const choice = this.lastResponse.choices[0];
        if (choice.deep_thinking_result) {
            return choice.deep_thinking_result;
        }
        
        return '无深度思考结果';
    }

    isLoading() {
        return this.isLoading;
    }

    clearHistory() {
        this.lastResponse = null;
        this.lastThinkingTime = 0;
        console.log('对话历史已清除');
    }
}

// 注册扩展
Scratch.extensions.register(new MoranKimiExtension());
```