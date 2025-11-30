```
(function(Scratch) {
    'use strict';
    
    class MorganMailExtension {
        constructor() {
            // 使用EmailJS作为邮件发送服务
            this.emailjsUserId = '';
            this.emailjsServiceId = '';
            this.emailjsTemplateId = '';
            this.isInitialized = false;
            
            this.colors = {
                primary: '#2E7D32',
                secondary: '#1B5E20',
                accent: '#4CAF50'
            };
        }
        
        getInfo() {
            return {
                id: 'Mmail',
                name: 'Morgan的邮件发送',
                color1: this.colors.primary,
                color2: this.colors.secondary,
                color3: this.colors.accent,
                docsURI: 'https://www.emailjs.com/docs/',
                blocks: [
                    {
                        opcode: 'initializeEmailService',
                        blockType: Scratch.BlockType.COMMAND,
                        text: '初始化邮件服务 UserID:[USER_ID] ServiceID:[SERVICE_ID] TemplateID:[TEMPLATE_ID]',
                        arguments: {
                            USER_ID: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'your_user_id'
                            },
                            SERVICE_ID: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'your_service_id'
                            },
                            TEMPLATE_ID: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'your_template_id'
                            }
                        }
                    },
                    '---',
                    {
                        opcode: 'sendEmail',
                        blockType: Scratch.BlockType.COMMAND,
                        text: '发送邮件 发件人:[FROM_EMAIL] 收件人:[TO_EMAIL] 主题:[SUBJECT] 内容:[CONTENT]',
                        arguments: {
                            FROM_EMAIL: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'sender@example.com'
                            },
                            TO_EMAIL: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'receiver@example.com'
                            },
                            SUBJECT: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'Hello from Scratch!'
                            },
                            CONTENT: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: '这是一封来自Scratch的测试邮件。'
                            }
                        }
                    },
                    {
                        opcode: 'sendEmailWithName',
                        blockType: Scratch.BlockType.COMMAND,
                        text: '发送邮件(带姓名) 发件人:[FROM_EMAIL] 发件人名:[FROM_NAME] 收件人:[TO_EMAIL] 收件人名:[TO_NAME] 主题:[SUBJECT] 内容:[CONTENT]',
                        arguments: {
                            FROM_EMAIL: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'sender@example.com'
                            },
                            FROM_NAME: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: '发件人'
                            },
                            TO_EMAIL: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'receiver@example.com'
                            },
                            TO_NAME: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: '收件人'
                            },
                            SUBJECT: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'Hello from Scratch!'
                            },
                            CONTENT: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: '这是一封来自Scratch的测试邮件。'
                            }
                        }
                    },
                    '---',
                    {
                        opcode: 'isServiceInitialized',
                        blockType: Scratch.BlockType.BOOLEAN,
                        text: '邮件服务已初始化?',
                        disableMonitor: true
                    },
                    {
                        opcode: 'getLastSendStatus',
                        blockType: Scratch.BlockType.REPORTER,
                        text: '上次发送状态',
                        disableMonitor: true
                    },
                    {
                        opcode: 'validateEmail',
                        blockType: Scratch.BlockType.BOOLEAN,
                        text: '邮箱地址 [EMAIL] 有效?',
                        arguments: {
                            EMAIL: {
                                type: Scratch.ArgumentType.STRING,
                                defaultValue: 'test@example.com'
                            }
                        }
                    }
                ]
            };
        }
        
        initializeEmailService(args) {
            this.emailjsUserId = args.USER_ID.trim();
            this.emailjsServiceId = args.SERVICE_ID.trim();
            this.emailjsTemplateId = args.TEMPLATE_ID.trim();
            
            if (this.emailjsUserId && this.emailjsServiceId && this.emailjsTemplateId) {
                this.isInitialized = true;
                console.log('邮件服务初始化成功');
                return '邮件服务初始化成功';
            } else {
                this.isInitialized = false;
                console.log('邮件服务初始化失败：参数不完整');
                return '初始化失败：请检查所有参数';
            }
        }
        
        isServiceInitialized() {
            return this.isInitialized;
        }
        
        getLastSendStatus() {
            return this.lastSendStatus || '尚未发送邮件';
        }
        
        validateEmail(args) {
            const email = args.EMAIL.trim();
            // 简单的邮箱格式验证
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            return emailRegex.test(email);
        }
        
        sendEmail(args) {
            if (!this.isInitialized) {
                this.lastSendStatus = '错误：请先初始化邮件服务';
                console.error(this.lastSendStatus);
                return this.lastSendStatus;
            }
            
            const fromEmail = args.FROM_EMAIL.trim();
            const toEmail = args.TO_EMAIL.trim();
            const subject = args.SUBJECT.trim();
            const content = args.CONTENT.trim();
            
            // 验证邮箱格式
            if (!this.validateEmail({EMAIL: fromEmail})) {
                this.lastSendStatus = '错误：发件人邮箱格式无效';
                return this.lastSendStatus;
            }
            
            if (!this.validateEmail({EMAIL: toEmail})) {
                this.lastSendStatus = '错误：收件人邮箱格式无效';
                return this.lastSendStatus;
            }
            
            // 使用EmailJS发送邮件
            const templateParams = {
                from_email: fromEmail,
                to_email: toEmail,
                subject: subject,
                message: content,
                from_name: 'Scratch用户',
                to_name: '收件人'
            };
            
            return this.sendEmailJS(templateParams);
        }
        
        sendEmailWithName(args) {
            if (!this.isInitialized) {
                this.lastSendStatus = '错误：请先初始化邮件服务';
                console.error(this.lastSendStatus);
                return this.lastSendStatus;
            }
            
            const fromEmail = args.FROM_EMAIL.trim();
            const fromName = args.FROM_NAME.trim();
            const toEmail = args.TO_EMAIL.trim();
            const toName = args.TO_NAME.trim();
            const subject = args.SUBJECT.trim();
            const content = args.CONTENT.trim();
            
            // 验证邮箱格式
            if (!this.validateEmail({EMAIL: fromEmail})) {
                this.lastSendStatus = '错误：发件人邮箱格式无效';
                return this.lastSendStatus;
            }
            
            if (!this.validateEmail({EMAIL: toEmail})) {
                this.lastSendStatus = '错误：收件人邮箱格式无效';
                return this.lastSendStatus;
            }
            
            // 使用EmailJS发送邮件
            const templateParams = {
                from_email: fromEmail,
                from_name: fromName,
                to_email: toEmail,
                to_name: toName,
                subject: subject,
                message: content
            };
            
            return this.sendEmailJS(templateParams);
        }
        
        sendEmailJS(templateParams) {
            // 动态加载EmailJS库
            return new Promise((resolve) => {
                // 检查是否已加载EmailJS
                if (typeof emailjs === 'undefined') {
                    // 动态加载EmailJS SDK
                    const script = document.createElement('script');
                    script.src = 'https://cdn.jsdelivr.net/npm/@emailjs/browser@3/dist/email.min.js';
                    script.onload = () => {
                        this.initializeAndSendEmail(templateParams, resolve);
                    };
                    script.onerror = () => {
                        this.lastSendStatus = '错误：加载邮件服务失败';
                        resolve(this.lastSendStatus);
                    };
                    document.head.appendChild(script);
                } else {
                    this.initializeAndSendEmail(templateParams, resolve);
                }
            });
        }
        
        initializeAndSendEmail(templateParams, resolve) {
            try {
                // 初始化EmailJS
                emailjs.init(this.emailjsUserId);
                
                // 发送邮件
                emailjs.send(this.emailjsServiceId, this.emailjsTemplateId, templateParams)
                    .then((response) => {
                        this.lastSendStatus = `成功：邮件已发送 (状态: ${response.status})`;
                        console.log('邮件发送成功:', response);
                        resolve(this.lastSendStatus);
                    })
                    .catch((error) => {
                        this.lastSendStatus = `错误：发送失败 - ${error.text || error.message}`;
                        console.error('邮件发送失败:', error);
                        resolve(this.lastSendStatus);
                    });
            } catch (error) {
                this.lastSendStatus = `错误：发送过程异常 - ${error.message}`;
                console.error('邮件发送异常:', error);
                resolve(this.lastSendStatus);
            }
        }
    }
    
    // 注册扩展
    Scratch.extensions.register(new MorganMailExtension());
})(Scratch);
```