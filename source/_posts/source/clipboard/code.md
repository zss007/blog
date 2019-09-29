---
title: clipboard 之源码
categories:
- source
---
最近有用到粘贴复制功能，调用了[clipboard.js](https://github.com/zenorocha/clipboard.js)库，对其原理很感兴趣。研读源码后开始总结，这篇主要是介绍其源码实现。
<!--more-->
### 一、clipboard.js
```
import ClipboardAction from './clipboard-action';
import Emitter from 'tiny-emitter';
import listen from 'good-listener';

// 继承 tiny-emitter，存在 on、once、emit、off 方法
class Clipboard extends Emitter {
    /**
     * @param {String|HTMLElement|HTMLCollection|NodeList} trigger
     * @param {Object} options
     */
    constructor(trigger, options) {
        super();

        this.resolveOptions(options);
        this.listenClick(trigger);
    }

    // 定义选项
    resolveOptions(options = {}) {
        this.action    = (typeof options.action    === 'function') ? options.action    : this.defaultAction;
        this.target    = (typeof options.target    === 'function') ? options.target    : this.defaultTarget;
        this.text      = (typeof options.text      === 'function') ? options.text      : this.defaultText;
        this.container = (typeof options.container === 'object')   ? options.container : document.body;
    }

    // 对触发器添加监听函数
    listenClick(trigger) {
        this.listener = listen(trigger, 'click', (e) => this.onClick(e));
    }

    // 触发器点击时触发
    onClick(e) {
        const trigger = e.delegateTarget || e.currentTarget;

        if (this.clipboardAction) {
            this.clipboardAction = null;
        }

        this.clipboardAction = new ClipboardAction({
            action    : this.action(trigger),
            target    : this.target(trigger),
            text      : this.text(trigger),
            container : this.container,
            trigger   : trigger,
            emitter   : this
        });
    }

    // 默认 action
    defaultAction(trigger) {
        return getAttributeValue('action', trigger);
    }

    // 默认 target
    defaultTarget(trigger) {
        const selector = getAttributeValue('target', trigger);

        if (selector) {
            return document.querySelector(selector);
        }
    }

    // 判断是否支持 execCommand 命令
    static isSupported(action = ['copy', 'cut']) {
        const actions = (typeof action === 'string') ? [action] : action;
        let support = !!document.queryCommandSupported;

        actions.forEach((action) => {
            support = support && !!document.queryCommandSupported(action);
        });

        return support;
    }

    // 默认 Text
    defaultText(trigger) {
        return getAttributeValue('text', trigger);
    }

    // 调用 destroy 移除事件监听，并调用 clipboardAction 的 destroy 方法
    destroy() {
        this.listener.destroy();

        if (this.clipboardAction) {
            this.clipboardAction.destroy();
            this.clipboardAction = null;
        }
    }
}


// 获取 data-clipboard-xxx 属性值
function getAttributeValue(suffix, element) {
    const attribute = `data-clipboard-${suffix}`;

    if (!element.hasAttribute(attribute)) {
        return;
    }

    return element.getAttribute(attribute);
}

module.exports = Clipboard;
```
### 二、clipboard-action.js
```
import select from 'select';

// 执行 copy 或 cut 操作
class ClipboardAction {
    /**
     * @param {Object} options
     */
    constructor(options) {
        this.resolveOptions(options);
        this.initSelection();
    }

    // 获取属性
    resolveOptions(options = {}) {
        this.action    = options.action;
        this.container = options.container;
        this.emitter   = options.emitter;
        this.target    = options.target;
        this.text      = options.text;
        this.trigger   = options.trigger;

        this.selectedText = '';
    }

    // 根据 text 和 target 选择不同的策略
    initSelection() {
        if (this.text) {
            this.selectFake();
        }
        else if (this.target) {
            this.selectTarget();
        }
    }

    // 创建一个 textarea 元素，为它设置值，并选中
    selectFake() {
        const isRTL = document.documentElement.getAttribute('dir') == 'rtl';

        this.removeFake();

        this.fakeHandlerCallback = () => this.removeFake();
        this.fakeHandler = this.container.addEventListener('click', this.fakeHandlerCallback) || true;

        this.fakeElem = document.createElement('textarea');
        // Prevent zooming on iOS
        this.fakeElem.style.fontSize = '12pt';
        // Reset box model
        this.fakeElem.style.border = '0';
        this.fakeElem.style.padding = '0';
        this.fakeElem.style.margin = '0';
        // Move element out of screen horizontally
        this.fakeElem.style.position = 'absolute';
        this.fakeElem.style[ isRTL ? 'right' : 'left' ] = '-9999px';
        // Move element to the same position vertically
        let yPosition = window.pageYOffset || document.documentElement.scrollTop;
        this.fakeElem.style.top = `${yPosition}px`;

        this.fakeElem.setAttribute('readonly', '');
        this.fakeElem.value = this.text;

        this.container.appendChild(this.fakeElem);

        this.selectedText = select(this.fakeElem);
        this.copyText();
    }

    // 移除添加的元素，只在点击后才移除，因为用户可能使用 `Ctrl+C` 来赋值当前选中的值
    removeFake() {
        if (this.fakeHandler) {
            this.container.removeEventListener('click', this.fakeHandlerCallback);
            this.fakeHandler = null;
            this.fakeHandlerCallback = null;
        }

        if (this.fakeElem) {
            this.container.removeChild(this.fakeElem);
            this.fakeElem = null;
        }
    }

    // 选中目标元素
    selectTarget() {
        this.selectedText = select(this.target);
        this.copyText();
    }

    // 在当前选中的元素上执行 copy 操作
    copyText() {
        let succeeded;

        try {
            succeeded = document.execCommand(this.action);
        }
        catch (err) {
            succeeded = false;
        }

        this.handleResult(succeeded);
    }

    // 触发 copy 操作的回调函数
    handleResult(succeeded) {
        this.emitter.emit(succeeded ? 'success' : 'error', {
            action: this.action,
            text: this.selectedText,
            trigger: this.trigger,
            clearSelection: this.clearSelection.bind(this)
        });
    }

    // 将焦点从目标元素移动到触发器，并移除 selection
    clearSelection() {
        if (this.trigger) {
            this.trigger.focus();
        }

        window.getSelection().removeAllRanges();
    }

    // 设置 action，并检测操作
    set action(action = 'copy') {
        this._action = action;

        if (this._action !== 'copy' && this._action !== 'cut') {
            throw new Error('Invalid "action" value, use either "copy" or "cut"');
        }
    }

    get action() {
        return this._action;
    }

    // 设置 target 并判断
    set target(target) {
        if (target !== undefined) {
            if (target && typeof target === 'object' && target.nodeType === 1) {
                if (this.action === 'copy' && target.hasAttribute('disabled')) {
                    throw new Error('Invalid "target" attribute. Please use "readonly" instead of "disabled" attribute');
                }

                if (this.action === 'cut' && (target.hasAttribute('readonly') || target.hasAttribute('disabled'))) {
                    throw new Error('Invalid "target" attribute. You can\'t cut text from elements with "readonly" or "disabled" attributes');
                }

                this._target = target;
            }
            else {
                throw new Error('Invalid "target" value, use a valid Element');
            }
        }
    }

    get target() {
        return this._target;
    }

    // 销毁
    destroy() {
        this.removeFake();
    }
}

module.exports = ClipboardAction;
```
