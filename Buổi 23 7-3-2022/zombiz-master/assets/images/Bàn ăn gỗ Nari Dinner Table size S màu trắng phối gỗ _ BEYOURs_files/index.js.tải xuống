OnCustomerUtils = {
  widgetId: 'oc-iframe-123456789',
  backgroundId: 'oc-iframe-background-123456789',
  commands: {
    SHOW_WIDGET: 'trigger:show-widget',
  },
  triggerHook: (type, data) => {
    if (!window.onCustomerSettings
      || !window.onCustomerSettings.hooks
      || !window.onCustomerSettings.hooks[type]) {
      return;
    }
    try { window.onCustomerSettings.hooks[type](data); } catch (error) { console.log('OnCustomer hook error', error); }
  },
  _checkAndBindToElement() {
    if (!window.onCustomerSettings || !window.onCustomerSettings.custom_launcher_selector) {
      return;
    }
    if (document && document.body) {
      doCheckAndBind();
    } else {
      window.onload = function () {
        doCheckAndBind();
      };
    }
    function clickHandle(evt) {
      let target = evt.target;
      while (target != null) {
        const isMatch = target.matches(window.onCustomerSettings.custom_launcher_selector);
        if (isMatch) {
          OnCustomer.command(OnCustomerUtils.commands.SHOW_WIDGET);
          return;
        }
        target = target.parentElement;
      }
    }
    function doCheckAndBind() {
      if (window.addEventListener) {
        document.body.addEventListener('click', clickHandle, true);
      } else {
        document.body.attachEvent('click', clickHandle, true);
      }
    }
  },
  extractWidgetDomain() {
    const scriptElem = document.getElementById('oc-chat-widget-bootstrap');
    const url = scriptElem.getAttribute('src');
    const regexp = /((https?:\/\/)[^/]+)/;
    const matchResult = regexp.exec(url);
    return matchResult && matchResult[1];
  },
  _keyStr: 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=',
  delayLoad(fn, timeout) {
    if (document.readyState === 'complete') {
      setTimeout(fn, timeout);
    } else {
      window.onload = function () {
        setTimeout(fn, timeout);
      };
    }
  },
  lazyLoad: ['6825ebb93887558d30d26b5c85731307', 'c71db54c0e752bf9cd8952003693213d', '0517d09289e39f36d8567d78eee0fca1'],
  encode(input) {
    let output = '';
    let chr1; let chr2; let chr3; let enc1; let enc2; let enc3; let
      enc4;
    let i = 0;

    input = this._utf8_encode(input);
    while (i < input.length) {
      chr1 = input.charCodeAt(i++);
      chr2 = input.charCodeAt(i++);
      chr3 = input.charCodeAt(i++);
      enc1 = chr1 >> 2;
      enc2 = ((chr1 & 3) << 4) | (chr2 >> 4);
      enc3 = ((chr2 & 15) << 2) | (chr3 >> 6);
      enc4 = chr3 & 63;
      if (isNaN(chr2)) {
        enc3 = enc4 = 64;
      } else if (isNaN(chr3)) {
        enc4 = 64;
      }
      output = output
        + this._keyStr.charAt(enc1) + this._keyStr.charAt(enc2)
        + this._keyStr.charAt(enc3) + this._keyStr.charAt(enc4);
    }
    return output;
  },
  _utf8_encode(string) {
    string = string.replace(/\r\n/g, '\n');
    let utftext = '';
    for (let n = 0; n < string.length; n++) {
      const c = string.charCodeAt(n);
      if (c < 128) {
        utftext += String.fromCharCode(c);
      } else if ((c > 127) && (c < 2048)) {
        utftext += String.fromCharCode((c >> 6) | 192);
        utftext += String.fromCharCode((c & 63) | 128);
      } else {
        utftext += String.fromCharCode((c >> 12) | 224);
        utftext += String.fromCharCode(((c >> 6) & 63) | 128);
        utftext += String.fromCharCode((c & 63) | 128);
      }
    }
    return utftext;
  },
};
OnCustomer = {
  ID_WIDGET: 'oc-widget',
  reset() {
    OnCustomerUtils.ready = false;
    const state = OnCustomer.state || {};
    if (state.invervalJob) {
      clearInterval(state.invervalJob);
    }
    OnCustomer.state = {};
    const currentWidget = document.getElementById(this.ID_WIDGET);
    if (currentWidget) {
      currentWidget.remove();
    }
  },
  reload() {
    OnCustomer.init(window.onCustomerSettings);
  },
  init(config, context) {
    OnCustomer.reset();
    const initTime = context && context.initTime ? context.initTime : new Date().getTime();
    if (OnCustomer.initTime && initTime < OnCustomer.initTime) {
      return;
    }
    OnCustomer.initTime = initTime;
    let oldHash;
    const _hashHandler = function () {
      oldHash = window.location.href;
      const detect = function () {
        if (oldHash === window.location.href) {
          return;
        }
        sendUrl();
        oldHash = window.location.href;
      };
      OnCustomer.state.invervalJob = setInterval(() => {
        // detect url and update to widget
        detect();
        // monitor main window focus status
        _checkFocus();
      }, 100);
    };
    _hashHandler();
    OnCustomerUtils._checkAndBindToElement();

    config = config || {};
    const domain = OnCustomerUtils.extractWidgetDomain();
    // make opener domain specification more strict for production environment
    const openerDomain = domain === 'https://widget.oncustomer.asia'
      ? 'https://livechat.oncustomer.asia'
      : '*';
    if (!domain) {
      return;
    }
    let widgetContainers;
    window.addEventListener('message', _handleMsg, false);
    window.addEventListener('resize', () => {
      _calculateWidgetContainerMaxHeight();
    });

    _processWidget(config, { initTime });
    function sendUrl() {
      _sendMessage({
        title: window.document.title,
        url: window.location.href,
        referrer: window.document.referrer,
        search: window.location.search,
        type: 'change-url',
      });
    }

    function _calculateWidgetContainerMaxHeight() {
      if (!widgetContainers) {
        return;
      }
      const clientRect = widgetContainers.getBoundingClientRect();
      widgetContainers.style.maxHeight = `${clientRect.bottom}px`;
      _sendMessage({ method: 'oc-iframe-max-height', maxHeight: clientRect.bottom });
    }

    function _sendMessage(data) {
      const iframe = document.getElementById(OnCustomerUtils.widgetId);
      if (!iframe) {
        return;
      }
      iframe.contentWindow.postMessage(data, '*');
    }

    function _initSelectionMode(data) {
      // load visual-selection script
      const selectionScriptElem = document.createElement('script');
      selectionScriptElem.id = 'oc-selection-script';
      selectionScriptElem.src = `${domain}/js/selection.js?sessionId=${data.sessionId
      }&domain=${domain
      }&openerDomain=${openerDomain}`;
      document.body.appendChild(selectionScriptElem);
    }

    /*
     * cross window custom communication logic
     */
    function _extractPath(object, path) {
      if (!path) {
        return object;
      }
      // normalization to array
      if (typeof path === 'string' || typeof path === 'number') {
        path = [path];
      }
      let idx = 0;
      let result = object;
      while (path[idx] && result) {
        const pathPart = path[idx];
        idx++;
        // 'attribute' type abbreviation
        if (typeof pathPart === 'string' || typeof pathPart === 'number') {
          result = result[pathPart];
          continue;
        }
        if (pathPart.type === 'attribute') {
          result = result[pathPart.name];
          continue;
        }
        if (pathPart.type === 'function') {
          if (!result[pathPart.name] || typeof result[pathPart.name] !== 'function') {
            result = null;
            continue;
          }
          let args = pathPart.args || [];
          args = args.map(_mapArg);
          result = result[pathPart.name].apply(result, args);
        }
      }
      return result;
    }

    function _mapArg(arg) {
      if (!arg.type) {
        return arg;
      }
      if (arg.type === 'object') {
        return arg.value;
      }
      if (arg.type == 'path') {
        const target = _getTargetElement(arg);
        return _extractPath(target, arg.path);
      }
      return null;
    }

    function _getTargetElement(eventConfig) {
      if (eventConfig.targetType === 'window') {
        return window;
      }
      if (eventConfig.targetType === 'document') {
        return document;
      }
      if (eventConfig.targetType === 'xpath') {
        return document.evaluate(eventConfig.targetPath, document).iterateNext();
      }
      return null;
    }

    function _getExtractTarget(event, pathConfig) {
      if (pathConfig.targetType) {
        return _getTargetElement(pathConfig);
      }
      return event;
    }

    function _handleWidgetRequest(request) {
      const payload = request.payload;
      let target;
      let result;
      try {
        target = _getTargetElement(payload);
        result = _extractPath(target, payload.path);
      } catch (ex) {
        let errorMessage = ex && ex.message;
        errorMessage = typeof errorMessage === 'string' ? errorMessage : 'unknown_error';
        _sendMessage({
          method: 'oc-query-response',
          response: {
            requestId: request.requestId,
            status: 'failure',
            error: errorMessage,
          },
        });
        return;
      }
      _sendMessage({
        method: 'oc-query-response',
        response: {
          result,
          status: 'success',
          requestId: request.requestId,
        },
      });
    }

    function _setupEventListener(listenerId, eventConfig) {
      let element;
      try {
        element = _getTargetElement(eventConfig);
      } catch (ex) {
        return;
      }
      if (!element) {
        return;
      }
      element.addEventListener(eventConfig.eventType, (e) => {
        // TODO: error handling
        const payload = { ...eventConfig.baseData || {} };
        if (eventConfig.paths) {
          eventConfig.paths.forEach((pathConfig) => {
            const extractTarget = _getExtractTarget(e, pathConfig);
            payload[pathConfig.key] = _extractPath(extractTarget, pathConfig.path);
          });
        }
        _sendMessage({
          method: 'oc-listen-event-triggered',
          payload,
          listenerId,
        });
      });
    }
    /* end of cross window communication logic */

    function _handleMsg(e) {
      if (!widgetContainers || !widgetContainers.classList) {
        return;
      }
      switch (e.data.method) {
        case 'oc-visual-selection-mode':
          // hide widget
          widgetContainers.style.display = 'none';
          _initSelectionMode(e.data);
          break;
        case 'oc-listen-event':
          _setupEventListener(e.data.listenerId, e.data.eventConfig);
          break;
        case 'oc-query':
          _handleWidgetRequest(e.data.request);
          break;
        case 'oc-chat-widget-change-height':
          const newClass = `oc-widget-${e.data.className}`;
          const oldClass = newClass === 'oc-widget-show' ? 'oc-widget-hide' : 'oc-widget-show';
          widgetContainers.classList.remove(oldClass);
          widgetContainers.classList.add(newClass);
          widgetContainers.style.height = null;
          widgetContainers.classList.remove('oc-widget-notif-new-message');
          widgetContainers.classList.remove('oc-widget-resize');
          if (e.data.className === 'show') {
            if (e.data.contextData && e.data.contextData.widgetSize) {
              widgetContainers.classList.add(`size-${e.data.contextData.widgetSize}`);
            }
            _calculateWidgetContainerMaxHeight();
            if (window.innerWidth < 450) {
              document.body.style.overflow = 'hidden';
            }
            OnCustomerUtils.triggerHook('onShow');
            updateBg(1);
          } else {
            if (e.data.contextData && e.data.contextData.widgetSize) {
              widgetContainers.classList.remove(`size-${e.data.contextData.widgetSize}`);
            }
            document.body.style.overflow = '';
            OnCustomerUtils.triggerHook('onHide');
            updateBg(0);
          }
          break;
        case 'oc-chat-widget-toggle-reading-size':
          if (e.data.className === 'oc-expand') {
            widgetContainers.classList.add('oc-widget-resize');
          } else if (e.data.className === 'oc-shrink') {
            setTimeout(() => {
              widgetContainers.classList.remove('oc-widget-resize');
            }, 300);
          }
          break;
        case 'hidden-widget':
          widgetContainers.classList.add('hidden-widget');
          updateBg(0);
          break;
        // TODO: Seem not used
        case 'remove-hidden-widget':
          widgetContainers.classList.remove('hidden-widget');
          break;
        case 'oc-chat-widget-notif-new-message':
          if (e.data.show) {
            widgetContainers.classList.add('oc-widget-notif-new-message');
            widgetContainers.style.height = e.data.height; // height to notice height
            updateBg(1);
          } else {
            widgetContainers.classList.remove('oc-widget-notif-new-message');
            widgetContainers.style.height = ''; // reset height
            updateBg(0);
          }
          _calculateWidgetContainerMaxHeight();
          break;
        case 'close-notif':
          widgetContainers.classList.remove('oc-widget-notif-new-message');
          widgetContainers.style.height = '';
          updateBg(0);
          break;
        case 'oc-show-overlay-image':
          var iframe = document.getElementById('oc-modal-frame');
          iframe.style.top = 0;
          iframe.style.display = 'block';
          iframe.style.left = 0;
          iframe.style.width = '100%';
          iframe.style.height = '100%';
          iframe.style.position = 'fixed';
          iframe.style['z-index'] = 2147483647;
          iframe.contentWindow.postMessage(e.data, domain);
          break;
        case 'oc-hide-overlay-image':
          var iframe = document.getElementById('oc-modal-frame');
          iframe.style.display = 'none';
          iframe.style.width = 0;
          iframe.style.height = 0;
          break;
        case 'oc-escape-pressed':
          var iframe = document.getElementById('oc-modal-frame');
          iframe.contentWindow.postMessage(e.data, domain);
          break;
        case 'oc-widget-ready':
          OnCustomerUtils.ready = true;
          break;
        case 'visitor:initialized':
          try {
            if (e.data && e.data.data) {
              OnCustomerUtils.triggerHook('onVisitor', e.data.data);
              OnCustomerUtils.visitor = e.data.data;
              console.log(OnCustomerUtils.visitor);
            }
          } catch (error) {}
          break;
        case 'oncustomer:feedback':
          initFeedback(e.data.data);
          break;
        default:
          break;
      }
    }
    function initFeedback(data) {
      OnCustomerUtils.feedback = data;
      if (data && Array.isArray(data.feedback)) {
        const srcUrl = document.getElementById('oc-chat-widget-bootstrap').src;
        const appToken = getUrlParameter(srcUrl, 'token');
        data.feedback.forEach(feedback => {
          const script = document.createElement('script');
          script.id = 'oc-chat-feedback-bootstrap-' + feedback._id;
          script.type = 'text/javascript';
          script.src = `${data.feedbackRoot}/js/index.js?id=${feedback._id}&token=${appToken}`;
          script.dataset.feedbackId = feedback._id;
          document.getElementsByTagName('head')[0].appendChild(script);
        });
      }
    }
    function updateBg(show) {
      // eslint-disable-next-line no-undef
      let bg = document.getElementById(OnCustomerUtils.backgroundId);
      const opacity = show === 1 ? 1 : 0;
      if (bg) {
        bg.style.opacity = opacity;
        return;
      }
      // eslint-disable-next-line no-undef
      bg = document.createElement('iframe');
      bg.id = OnCustomerUtils.backgroundId;
      bg.style.opacity = show;
      bg.classList.add('oc-radial-background');
      if (!OnCustomerUtils.wrapper) {
        return;
      }
      OnCustomerUtils.wrapper.prepend(bg);
    }

    function _getConfigParams(_config) {
      if (!_config) {
        return;
      }
      return OnCustomerUtils.encode(JSON.stringify(_config));
    }

    function _processWidget(config, context) {
      const srcUrl = document.getElementById('oc-chat-widget-bootstrap').src;
      const appToken = getUrlParameter(srcUrl, 'token');
      const widgetId = 'oc-widget';
      const lang = getUrlParameter(srcUrl, 'lang') || 'vi';
      const wrapper = document.createElement('div');
      OnCustomerUtils.wrapper = wrapper;
      wrapper.id = widgetId;
      wrapper.classList.add('oc-widget');
      if (window.onCustomerSettings && window.onCustomerSettings.custom_launcher_selector) {
        wrapper.classList.add('oc-cl');
      }
      widgetContainers = wrapper;
      const configParams = _getConfigParams(config);
      let iframe = document.getElementById(OnCustomerUtils.widgetId);
      const data = JSON.stringify({
        title: window.document.title,
        referrer: window.document.referrer,
        url: window.location.href,
        search: window.location.search,
      });
      let src = `${domain
      }/livechat/?widgetId=${widgetId
      }&appToken=${appToken
      }&lang=${lang
      }&ocdata=${OnCustomerUtils.encode(data)}`;
      if (configParams) {
        src += `&ocvisitordata=${configParams}`;
      }
      const botSimulationId = getUrlParameter(
        document.getElementById('oc-chat-widget-bootstrap').src,
        'botSimulationId',
      );
      if (botSimulationId) {
        src += `&botSimulationId=${botSimulationId}`;
      }
      if (!iframe || iframe.length === 0) {
        _appendCSS(appToken);
        iframe = document.createElement('iframe');
        iframe.id = OnCustomerUtils.widgetId;
        iframe.style['border-width'] = '0px';
        iframe.setAttribute('allowFullScreen', '');
        iframe.style.width = '100%';
        iframe.style.height = '100%';
        iframe.classList.add('widget-iframe');
        iframe.scrolling = 'no';
        iframe.src = src;
        wrapper.appendChild(iframe);
        _addMainFrame(wrapper, appToken, context);
      } else {
        iframe.src = src;
      }
    }
    function _addMainFrame(frame, appToken, context) {
      const readyStateCheckInterval = setInterval(() => {
        if (!document || !document.body) {
          return;
        }
        clearInterval(readyStateCheckInterval);
        if (OnCustomerUtils.lazyLoad.indexOf(appToken) >= 0) {
          OnCustomerUtils.delayLoad(() => {
            _addFrames(frame, context);
          }, 2000);
        } else {
          _addFrames(frame, context);
        }
      }, 10);
    }

    function _addFrames(frame, context = {}) {
      if (context.initTime !== OnCustomer.initTime) {
        return;
      }
      _addIframeToBody(frame);
      _addModalFrameToBody();
    }

    function _addIframeToBody(wrapper, cb) {
      document.body.appendChild(wrapper);
      if (cb) cb();
      if (window._onOnCustomerLoaded && typeof window._onOnCustomerLoaded === 'function') {
        window._onOnCustomerLoaded();
      }
    }

    function _addModalFrameToBody() {
      let iframe = document.getElementById('oc-modal-frame');
      if (iframe) {
        return;
      }
      iframe = document.createElement('iframe');
      iframe.id = 'oc-modal-frame';
      iframe.src = `${domain}/modal.html`;
      iframe.frameBorder = 0;
      _addIframeToBody(iframe);
    }

    function _appendCSS(appToken) {
      _appendCSSUrl(`${domain}/style/widget-style.css`);
      _appendCSSUrl(`https://s3-ap-southeast-1.amazonaws.com/oc.auto-generated/css/${appToken}.css?t=${new Date().getTime()}`);
    }

    function _appendCSSUrl(url) {
      const link = document.createElement('link');
      link.rel = 'stylesheet';
      link.type = 'text/css';
      link.href = url;
      document.head.appendChild(link);
    }

    function getUrlParameter(url, name) {
      name = name.replace(/[\[]/, '\\[').replace(/[\]]/, '\\]');
      const regex = new RegExp(`[\\?&]${name}=([^&#]*)`);
      const results = regex.exec(url || window.location.search);
      return results === null
        ? ''
        : decodeURIComponent(results[1].replace(/\+/g, ' '));
    }

    function _checkFocus() {
      if (this.isFocus === document.hasFocus()) {
        return;
      }
      this.isFocus = document.hasFocus();
      _sendMessage({
        value: this.isFocus,
        type: 'browser:focus-change',
      });
    }
    // window.opener could be our own app opening the widget under some features
    if (window.opener) {
      try {
        window.opener.postMessage({
          method: 'oc-widget-loaded',
          mode: getOcMode(),
        }, openerDomain);
      } catch (e) {
        console.log('oc-widget-loaded postMessage failed', e);
      }
    }
    function getOcMode() {
      const ocMode = getUrlParameter(window.location.href, 'ocMode') || sessionStorage.getItem('ocMode');
      sessionStorage.setItem('ocMode', ocMode);
      return ocMode;
    }
  },
  logout() {
    window.onCustomerSettings = {};
    OnCustomer.init(null, { initTime: new Date().getTime() });
  },
  // trigger:show-widget, init-message, trigger:hide-widget
  command(type, data) {
    const iframe = document.getElementById(OnCustomerUtils.widgetId);
    if (!iframe || !OnCustomerUtils.ready) {
      return setTimeout(() => OnCustomer.command(type, data), 100);
    }
    iframe.contentWindow.postMessage(
      { type, content: data }, '*',
    );
  },
  showMessage(data) {
    OnCustomer.command('init-message', data);
  },
  feedback(triggerName) {
    document.querySelectorAll('iframe[id^=oc-iframe-feedback]').forEach(iframe => {
      iframe.contentWindow.postMessage({
        method: 'oc-feedback-trigger',
        triggerName,
      }, '*');
    });
  },
};
try {
  OnCustomer.init(window.onCustomerSettings, { initTime: new Date().getTime() });
} catch (ex) {
  console.log('OnCustomer', ex);
}
