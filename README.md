<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>网站嵌入工具 - 全屏白屏修复版</title>
    <!-- 外部依赖 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.5.1/css/all.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/qrcode@1.5.3/build/qrcode.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#165DFF',
                        secondary: '#6B7280',
                        success: '#10B981',
                        warning: '#F59E0B',
                        danger: '#EF4444',
                        dark: '#1F2937',
                        light: '#F3F4F6'
                    },
                    fontFamily: {
                        inter: ['Inter', 'system-ui', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .glass-effect {
                backdrop-filter: blur(12px);
                background-color: rgba(255, 255, 255, 0.85);
            }
            .dark .glass-effect {
                background-color: rgba(31, 41, 55, 0.85);
            }
            .tab-active {
                @apply border-b-2 border-primary text-primary font-medium;
            }
            .transition-custom {
                transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            }
            /* 修复全屏白屏：iframe全屏样式，透明背景，铺满屏幕 */
            .fullscreen-iframe {
                width: 100% !important;
                height: 100% !important;
                border: 0 !important;
                margin: 0 !important;
                padding: 0 !important;
                border-radius: 0 !important;
                box-shadow: none !important;
                overflow: auto !important;
                background: transparent !important; /* 关键：透明背景，显示网页原生背景 */
            }
            /* 悬浮控制台 */
            .float-ctrl-wrapper {
                position: fixed;
                right: 30px;
                bottom: 30px;
                z-index: 9999;
                transition: all 0.3s ease;
            }
            .float-ctrl-btn {
                width: 64px;
                height: 64px;
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                cursor: move;
                box-shadow: 0 6px 25px rgba(0,0,0,0.18);
                transition: all 0.3s ease;
                user-select: none;
                touch-action: none;
            }
            .dark .float-ctrl-btn {
                box-shadow: 0 6px 25px rgba(0,0,0,0.4);
            }
            .float-ctrl-btn:hover {
                transform: scale(1.05);
            }
            .float-ctrl-content {
                max-height: 0;
                overflow: hidden;
                opacity: 0;
                transition: max-height 0.5s ease-in-out, opacity 0.4s ease;
                border-radius: 20px 20px 0 0;
                box-shadow: 0 -4px 20px rgba(0,0,0,0.12);
                margin-bottom: 12px;
                max-width: 90vw;
                width: 800px;
                background: #fff;
            }
            .dark .float-ctrl-content {
                background: #1F2937;
                box-shadow: 0 -4px 20px rgba(0,0,0,0.35);
            }
            .float-ctrl-content.expand {
                max-height: 85vh;
                opacity: 1;
                overflow-y: auto;
                overflow-x: hidden;
            }
            .iframe-container {
                position: relative;
                border-radius: 2xl;
                box-shadow: 0 4px 20px rgba(0,0,0,0.08);
                overflow: hidden;
                width: 100%;
                transition: all 0.3s ease;
            }
            .dark .iframe-container {
                box-shadow: 0 4px 20px rgba(0,0,0,0.25);
            }
        }
    </style>
    <style>
        /* 全局样式 */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Inter', system-ui, sans-serif;
            background-color: #F3F4F6;
            min-height: 100vh;
            transition: all 0.3s ease;
        }
        .dark {
            background-color: #111827;
            color: #F3F4F6;
        }
        .dark .bg-white {
            background-color: #1F2937;
        }
        .dark .bg-gray-50 {
            background-color: #111827;
        }
        .dark .bg-gray-100 {
            background-color: #374151;
        }
        .dark .bg-gray-200 {
            background-color: #4B5563;
        }
        .dark .text-gray-900 {
            color: #F3F4F6;
        }
        .dark .text-gray-600 {
            color: #D1D5DB;
        }
        .dark .text-gray-400 {
            color: #9CA3AF;
        }
        .dark .border-gray-200 {
            border-color: #374151;
        }
        .dark .border-gray-300 {
            border-color: #4B5563;
        }
        .dark .hover:bg-gray-200:hover {
            background-color: #5B6472;
        }
        .dark .hover:bg-gray-300:hover {
            background-color: #6B7280;
        }
        .dark .placeholder-gray-400::placeholder {
            color: #6B7280;
        }
        /* 滚动条优化 */
        ::-webkit-scrollbar {
            width: 7px;
            height: 7px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb {
            background: #c1c1c1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #a8a8a8;
        }
        .dark ::-webkit-scrollbar-track {
            background: #374151;
        }
        .dark ::-webkit-scrollbar-thumb {
            background: #6b7280;
        }
        .dark ::-webkit-scrollbar-thumb:hover {
            background: #9CA3AF;
        }
        /* 全屏模式：隐藏页面滚动条，保留iframe滚动条 */
        body.fullscreen-mode {
            overflow: hidden !important;
        }
        :fullscreen, ::-webkit-full-screen, ::-moz-full-screen {
            overflow: hidden !important;
        }
        /* 二维码容器 */
        #qr-wrap {
            background: #ffffff;
            padding: 24px;
            border-radius: 12px;
            text-align: center;
            width: fit-content;
            margin: 0 auto;
        }
        .dark #qr-wrap {
            background: #1F2937;
        }
        /* 加载/错误遮罩层 */
        .overlay {
            position: absolute;
            inset: 0;
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 99999 !important;
            background-color: rgba(255,255,255,0.92);
        }
        .dark .overlay {
            background-color: rgba(17,24,39,0.92);
        }
        /* 弹窗样式 */
        .modal {
            position: fixed;
            inset: 0;
            background-color: rgba(0,0,0,0.5);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 999999;
            padding: 20px;
        }
        .dark .modal {
            background-color: rgba(0,0,0,0.7);
        }
        .modal-content {
            background: #fff;
            border-radius: 20px;
            width: 100%;
            max-width: 500px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.15);
        }
        .dark .modal-content {
            background: #1F2937;
            box-shadow: 0 10px 30px rgba(0,0,0,0.4);
        }
    </style>
</head>
<body class="bg-gray-50 transition-custom">
    <!-- 主容器：非全屏显示所有内容，全屏时隐藏非网页内容 -->
    <div id="normal-container" class="w-full min-h-screen transition-custom py-6 px-4 sm:px-6 lg:px-8">
        <!-- 顶部导航栏 -->
        <header class="glass-effect border-b border-gray-200 sticky top-0 z-50 w-full mb-8">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="flex justify-between items-center h-16">
                    <div class="flex items-center space-x-2">
                        <i class="fas fa-link text-primary text-2xl"></i>
                        <h1 class="text-xl font-bold text-gray-900">网站嵌入工具</h1>
                    </div>
                    <div class="flex items-center space-x-4">
                        <button id="theme-toggle" class="p-2 rounded-full hover:bg-gray-200 transition-custom">
                            <i class="fas fa-moon text-gray-600"></i>
                        </button>
                        <button id="help-btn" class="p-2 rounded-full hover:bg-gray-200 transition-custom">
                            <i class="fas fa-question-circle text-gray-600"></i>
                        </button>
                    </div>
                </div>
            </div>
        </header>

        <!-- 主要功能区域 -->
        <main class="max-w-7xl mx-auto">
            <!-- URL输入区 -->
            <div id="url-section" class="mb-6 bg-white rounded-2xl shadow-lg p-6 border border-gray-100">
                <div class="flex flex-col md:flex-row gap-4 items-center">
                    <div class="flex-1 relative">
                        <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                            <i class="fas fa-globe text-gray-400"></i>
                        </div>
                        <input 
                            type="url" 
                            id="website-url" 
                            placeholder="请输入要嵌入的网站URL (例如: https://www.baidu.com)"
                            class="block w-full pl-10 pr-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-primary/50 focus:border-primary transition-custom dark:bg-dark dark:text-white placeholder-gray-400 outline-none"
                        >
                    </div>
                    <div class="flex space-x-3 shrink-0">
                        <button id="qr-btn" class="bg-gray-200 text-gray-700 px-4 py-3 rounded-xl hover:bg-gray-300 transition-custom font-medium" title="生成当前页面二维码">
                            <i class="fas fa-qrcode"></i>
                        </button>
                        <button id="go-btn" class="bg-primary text-white px-6 py-3 rounded-xl hover:bg-primary/90 transition-custom font-medium flex items-center space-x-2 shrink-0">
                            <i class="fas fa-arrow-right"></i>
                            <span>访问</span>
                        </button>
                        <button id="refresh-btn" class="bg-gray-200 text-gray-700 px-6 py-3 rounded-xl hover:bg-gray-300 transition-custom font-medium flex items-center space-x-2 shrink-0">
                            <i class="fas fa-sync-alt"></i>
                            <span>刷新</span>
                        </button>
                    </div>
                </div>
            </div>

            <!-- 检索栏+历史/收藏 -->
            <div id="search-section" class="mb-6 bg-white rounded-2xl shadow-lg p-4 border border-gray-100">
                <div class="relative mb-4">
                    <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                        <i class="fas fa-search text-gray-400"></i>
                    </div>
                    <input 
                        type="text" 
                        id="search-input" 
                        placeholder="检索历史记录/收藏夹（标题/URL）"
                        class="block w-full pl-10 pr-4 py-2 border border-gray-300 rounded-xl focus:ring-2 focus:ring-primary/50 focus:border-primary transition-custom dark:bg-dark dark:text-white placeholder-gray-400 outline-none"
                    >
                </div>
                <div class="flex border-b border-gray-200">
                    <button id="history-tab" class="tab-active py-2 px-4 font-medium transition-custom">
                        <i class="fas fa-history mr-2"></i>历史记录
                        <span id="history-count" class="bg-primary/10 text-primary text-xs px-2 py-0.5 rounded-full">0</span>
                    </button>
                    <button id="fav-tab" class="py-2 px-4 font-medium text-gray-600 hover:text-primary transition-custom">
                        <i class="fas fa-star mr-2"></i>收藏夹
                        <span id="fav-count" class="bg-primary/10 text-primary text-xs px-2 py-0.5 rounded-full">0</span>
                    </button>
                </div>
                <div id="history-box" class="mt-4 max-h-40 overflow-y-auto space-y-2">
                    <div class="text-center text-gray-500 py-4">暂无历史记录</div>
                </div>
                <div id="fav-box" class="mt-4 max-h-40 overflow-y-auto space-y-2 hidden">
                    <div class="text-center text-gray-500 py-4">暂无收藏</div>
                </div>
                <div id="batch-op" class="mt-4 flex justify-between hidden">
                    <button id="clear-history" class="text-sm text-danger hover:underline">清空历史记录</button>
                    <button id="add-fav" class="text-sm text-primary hover:underline flex items-center">
                        <i class="fas fa-star mr-1"></i>添加当前页面到收藏
                    </button>
                </div>
            </div>

            <!-- 控制工具栏 -->
            <div id="control-section" class="mb-6 bg-white rounded-2xl shadow-lg p-4 border border-gray-100">
                <div class="flex flex-wrap items-center gap-3">
                    <button id="back-btn" class="flex items-center space-x-2 px-4 py-2 rounded-lg bg-gray-100 hover:bg-gray-200 transition-custom text-gray-700 disabled:opacity-50 disabled:cursor-not-allowed">
                        <i class="fas fa-arrow-left"></i>
                        <span>后退</span>
                    </button>
                    <button id="forward-btn" class="flex items-center space-x-2 px-4 py-2 rounded-lg bg-gray-100 hover:bg-gray-200 transition-custom text-gray-700 disabled:opacity-50 disabled:cursor-not-allowed">
                        <i class="fas fa-arrow-right"></i>
                        <span>前进</span>
                    </button>
                    <button id="enter-full" class="flex items-center space-x-2 px-4 py-2 rounded-lg bg-gray-100 hover:bg-gray-200 transition-custom text-gray-700">
                        <i class="fas fa-expand"></i>
                        <span>全屏</span>
                    </button>
                    <div class="h-6 w-px bg-gray-300 mx-2 hidden sm:block"></div>
                    <div class="flex items-center space-x-2 text-sm text-gray-600 flex-1 sm:flex-auto">
                        <i class="fas fa-info-circle"></i>
                        <span>当前页面: <span id="current-page" class="font-medium text-primary break-all">未选择网站</span></span>
                    </div>
                </div>
            </div>

            <!-- 网页嵌入容器：全屏时保留此容器，其他内容隐藏 -->
            <div id="iframe-container" class="iframe-container bg-white border border-gray-100">
                <!-- 加载状态 -->
                <div id="loading" class="overlay hidden">
                    <div class="text-center">
                        <div class="inline-block animate-spin rounded-full h-12 w-12 border-b-2 border-primary mb-4"></div>
                        <p class="text-gray-600 dark:text-gray-300">正在加载网站...</p>
                    </div>
                </div>
                <!-- 错误提示 -->
                <div id="error" class="overlay hidden">
                    <div class="text-center max-w-md px-4">
                        <i class="fas fa-exclamation-triangle text-4xl text-warning mb-4"></i>
                        <h3 class="text-xl font-bold text-gray-900 dark:text-white mb-2">加载失败</h3>
                        <p id="error-txt" class="text-gray-600 dark:text-gray-300 mb-4">无法加载指定的网站，请检查URL是否正确或尝试其他网站。</p>
                        <button id="retry-btn" class="bg-primary text-white px-6 py-2 rounded-lg hover:bg-primary/90 transition-custom">
                            重试
                        </button>
                    </div>
                </div>
                <!-- 初始提示 -->
                <div id="initial-tip" class="absolute inset-0 bg-gray-50 dark:bg-dark flex items-center justify-center z-30">
                    <div class="text-center max-w-md px-4">
                        <i class="fas fa-link text-6xl text-primary/30 mb-6"></i>
                        <h3 class="text-2xl font-bold text-gray-900 dark:text-white mb-4">网站嵌入工具</h3>
                        <p class="text-gray-600 dark:text-gray-300 mb-6">输入URL后，所有超链接将在工具内打开，无需跳转新标签！</p>
                        <div class="bg-primary/10 rounded-lg p-4 text-left dark:bg-primary/20">
                            <h4 class="font-semibold text-gray-900 dark:text-white mb-2">使用示例:</h4>
                            <ul class="text-sm text-gray-600 dark:text-gray-300 space-y-1">
                                <li>• https://www.baidu.com</li>
                                <li>• https://www.github.com</li>
                                <li>• https://www.zhihu.com</li>
                            </ul>
                        </div>
                    </div>
                </div>
                <!-- 核心iframe：非全屏自适应，全屏铺满 -->
                <iframe 
                    id="main-iframe" 
                    name="embed-frame"
                    class="w-full h-full border-0"
                    sandbox="allow-scripts allow-same-origin allow-popups allow-forms allow-modals allow-top-navigation-by-user-activation"
                    referrerpolicy="no-referrer"
                    src="about:blank"
                ></iframe>
            </div>
        </main>

        <!-- 页脚 -->
        <footer id="footer-section" class="bg-white border-t border-gray-200 py-6 mt-12 dark:bg-dark dark:border-gray-700">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 text-center">
                <p class="text-gray-600 dark:text-gray-300 text-sm">
                    © 2025 网站嵌入工具 | 安全提示：仅访问您信任的网站 | 部分网站因安全策略无法嵌入
                </p>
            </div>
        </footer>
    </div>

    <!-- 全屏悬浮控制台：全屏时显示 -->
    <div id="float-ctrl" class="float-ctrl-wrapper hidden">
        <div id="float-ctrl-content" class="float-ctrl-content p-6">
            <div class="space-y-6">
                <div class="relative">
                    <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                        <i class="fas fa-globe text-gray-400"></i>
                    </div>
                    <input 
                        type="url" 
                        id="float-website-url" 
                        placeholder="输入网站URL访问"
                        class="block w-full pl-10 pr-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-primary/50 focus:border-primary transition-custom dark:bg-dark dark:text-white placeholder-gray-400 outline-none"
                    >
                </div>
                <div class="flex space-x-3 w-full">
                    <button id="float-qr-btn" class="flex-1 bg-gray-200 text-gray-700 py-3 rounded-xl hover:bg-gray-300 transition-custom font-medium">
                        <i class="fas fa-qrcode mr-2"></i>二维码
                    </button>
                    <button id="float-go-btn" class="flex-2 bg-primary text-white py-3 rounded-xl hover:bg-primary/90 transition-custom font-medium">
                        <i class="fas fa-arrow-right mr-2"></i>访问
                    </button>
                    <button id="float-refresh-btn" class="flex-1 bg-gray-200 text-gray-700 py-3 rounded-xl hover:bg-gray-300 transition-custom font-medium">
                        <i class="fas fa-sync-alt mr-2"></i>刷新
                    </button>
                </div>
                <div class="flex space-x-3 w-full">
                    <button id="float-back-btn" class="flex-1 bg-gray-200 text-gray-700 py-2 rounded-lg hover:bg-gray-300 transition-custom font-medium disabled:opacity-50">
                        <i class="fas fa-arrow-left mr-1"></i>后退
                    </button>
                    <button id="float-forward-btn" class="flex-1 bg-gray-200 text-gray-700 py-2 rounded-lg hover:bg-gray-300 transition-custom font-medium disabled:opacity-50">
                        <i class="fas fa-arrow-right mr-1"></i>前进
                    </button>
                    <button id="float-exit-full" class="flex-1 bg-danger text-white py-2 rounded-lg hover:bg-danger/90 transition-custom font-medium">
                        <i class="fas fa-compress mr-1"></i>退出全屏
                    </button>
                </div>
                <div class="space-y-4">
                    <div class="relative">
                        <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                            <i class="fas fa-search text-gray-400"></i>
                        </div>
                        <input 
                            type="text" 
                            id="float-search-input" 
                            placeholder="检索历史/收藏"
                            class="block w-full pl-10 pr-4 py-2 border border-gray-300 rounded-xl focus:ring-2 focus:ring-primary/50 focus:border-primary transition-custom dark:bg-dark dark:text-white placeholder-gray-400 outline-none"
                        >
                    </div>
                    <div class="flex border-b border-gray-200">
                        <button id="float-history-tab" class="tab-active py-2 px-4 font-medium transition-custom w-1/2 text-center">
                            <i class="fas fa-history mr-1"></i>历史
                            <span id="float-history-count" class="bg-primary/10 text-primary text-xs px-2 py-0.5 rounded-full">0</span>
                        </button>
                        <button id="float-fav-tab" class="py-2 px-4 font-medium text-gray-600 hover:text-primary transition-custom w-1/2 text-center">
                            <i class="fas fa-star mr-1"></i>收藏
                            <span id="float-fav-count" class="bg-primary/10 text-primary text-xs px-2 py-0.5 rounded-full">0</span>
                        </button>
                    </div>
                    <div id="float-history-box" class="max-h-32 overflow-y-auto space-y-2">
                        <div class="text-center text-gray-500 py-2">暂无历史记录</div>
                    </div>
                    <div id="float-fav-box" class="max-h-32 overflow-y-auto space-y-2 hidden">
                        <div class="text-center text-gray-500 py-2">暂无收藏</div>
                    </div>
                </div>
                <div class="flex justify-between items-center pt-4 border-t border-gray-200">
                    <button id="float-theme-toggle" class="p-2 rounded-full hover:bg-gray-200 transition-custom">
                        <i class="fas fa-moon text-gray-600"></i>
                    </button>
                    <button id="float-help-btn" class="p-2 rounded-full hover:bg-gray-200 transition-custom">
                        <i class="fas fa-question-circle text-gray-600"></i>
                    </button>
                </div>
            </div>
        </div>
        <div id="float-ctrl-btn" class="float-ctrl-btn bg-primary text-white">
            <i class="fas fa-cog text-2xl"></i>
        </div>
    </div>

    <!-- 弹窗：历史编辑/收藏/二维码/帮助 -->
    <div id="edit-history-modal" class="modal hidden">
        <div class="modal-content p-6">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-lg font-bold text-gray-900 dark:text-white">编辑历史记录名称</h3>
                <button id="close-edit-history" class="text-gray-400 hover:text-gray-600 dark:hover:text-gray-200">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <div class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">网页名称</label>
                    <input type="text" id="edit-history-name" class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 dark:bg-dark rounded-lg focus:ring-primary focus:border-primary outline-none">
                </div>
                <input type="hidden" id="edit-history-id">
                <div class="flex space-x-3 mt-6">
                    <button id="edit-history-cancel" class="flex-1 px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 transition-custom">
                        取消
                    </button>
                    <button id="edit-history-save" class="flex-1 px-4 py-2 bg-primary text-white rounded-lg hover:bg-primary/90 transition-custom">
                        保存
                    </button>
                </div>
            </div>
        </div>
    </div>

    <div id="fav-modal" class="modal hidden">
        <div class="modal-content p-6">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-lg font-bold text-gray-900 dark:text-white" id="fav-modal-title">添加收藏</h3>
                <button id="close-fav" class="text-gray-400 hover:text-gray-600 dark:hover:text-gray-200">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <div class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">收藏名称</label>
                    <input type="text" id="fav-name" class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 dark:bg-dark rounded-lg focus:ring-primary focus:border-primary outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">网站URL</label>
                    <input type="url" id="fav-url" class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 dark:bg-dark rounded-lg focus:ring-primary focus:border-primary outline-none">
                </div>
                <input type="hidden" id="fav-id">
                <div class="flex space-x-3 mt-6">
                    <button id="fav-cancel" class="flex-1 px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 transition-custom">
                        取消
                    </button>
                    <button id="fav-save" class="flex-1 px-4 py-2 bg-primary text-white rounded-lg hover:bg-primary/90 transition-custom">
                        保存
                    </button>
                </div>
            </div>
        </div>
    </div>

    <div id="qr-modal" class="modal hidden">
        <div class="modal-content p-6">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-lg font-bold text-gray-900 dark:text-white">网页二维码</h3>
                <button id="close-qr" class="text-gray-400 hover:text-gray-600 dark:hover:text-gray-200">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <div class="space-y-4">
                <div class="flex justify-center gap-4">
                    <button id="qr-minus" class="bg-gray-200 dark:bg-gray-700 px-3 py-1 rounded hover:bg-gray-300 transition-custom" title="缩小">
                        <i class="fas fa-minus"></i>
                    </button>
                    <button id="qr-plus" class="bg-gray-200 dark:bg-gray-700 px-3 py-1 rounded hover:bg-gray-300 transition-custom" title="放大">
                        <i class="fas fa-plus"></i>
                    </button>
                    <button id="qr-download" class="bg-primary text-white px-4 py-1 rounded hover:bg-primary/90 transition-custom">
                        <i class="fas fa-download mr-1"></i>下载
                    </button>
                </div>
                <div id="qr-wrap">
                    <div id="qr-img"></div>
                    <p id="qr-title" class="mt-3 text-gray-800 dark:text-gray-200 font-medium break-all text-center"></p>
                </div>
            </div>
        </div>
    </div>

    <div id="help-modal" class="modal hidden">
        <div class="modal-content p-6 max-w-2xl">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-lg font-bold text-gray-900 dark:text-white">使用帮助</h3>
                <button id="close-help" class="text-gray-400 hover:text-gray-600 dark:hover:text-gray-200">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <div class="space-y-4 text-gray-600 dark:text-gray-300">
                <div>
                    <h4 class="font-semibold text-gray-900 dark:text-white mb-2">核心功能</h4>
                    <p>全屏网页无框铺满、悬浮可拖动控制台、历史记录名称编辑、二维码生成下载、屏幕智能布局适配</p>
                </div>
                <div>
                    <h4 class="font-semibold text-gray-900 dark:text-white mb-2">全屏操作</h4>
                    <ul class="list-disc list-inside space-y-1">
                        <li>点击「全屏」后网页铺满屏幕，保留原生滚动条</li>
                        <li>右下角圆形按钮可按住拖动，点击拉出/回缩控制台</li>
                        <li>控制台包含所有功能，点击「退出全屏」恢复原界面</li>
                    </ul>
                </div>
                <div>
                    <h4 class="font-semibold text-gray-900 dark:text-white mb-2">历史/收藏操作</h4>
                    <ul class="list-disc list-inside space-y-1">
                        <li>历史记录：点击「编辑」可修改网页名称，URL不可改</li>
                        <li>收藏夹：支持添加/编辑/删除，自定义名称和URL</li>
                        <li>全局检索：输入关键词，实时筛选历史/收藏的标题/URL</li>
                    </ul>
                </div>
                <div>
                    <h4 class="font-semibold text-gray-900 dark:text-white mb-2">二维码功能</h4>
                    <ul class="list-disc list-inside space-y-1">
                        <li>加载网站后点击「二维码」生成当前页面码</li>
                        <li>「+/-」可缩放二维码（0.5-2倍）</li>
                        <li>「下载」将标题+二维码合成一张PNG图保存</li>
                    </ul>
                </div>
                <div>
                    <h4 class="font-semibold text-gray-900 dark:text-white mb-2">常见问题</h4>
                    <ul class="list-disc list-inside space-y-1">
                        <li>部分网站（淘宝/微信/抖音/B站）因防嵌入策略无法加载，属正常现象</li>
                        <li>全屏白屏：已修复，现在全屏时网页会正常显示</li>
                        <li>二维码生成失败：确保网络通畅，URL有效</li>
                    </ul>
                </div>
            </div>
        </div>
    </div>

    <script>
        // ========== 全局常量&DOM元素 ==========
        const DOM = {
            // 核心容器
            normalContainer: document.getElementById('normal-container'),
            iframeContainer: document.getElementById('iframe-container'),
            mainIframe: document.getElementById('main-iframe'),
            urlSection: document.getElementById('url-section'),
            searchSection: document.getElementById('search-section'),
            controlSection: document.getElementById('control-section'),
            footerSection: document.getElementById('footer-section'),
            // 非全屏功能元素
            websiteUrl: document.getElementById('website-url'),
            goBtn: document.getElementById('go-btn'),
            refreshBtn: document.getElementById('refresh-btn'),
            backBtn: document.getElementById('back-btn'),
            forwardBtn: document.getElementById('forward-btn'),
            enterFullBtn: document.getElementById('enter-full'),
            qrBtn: document.getElementById('qr-btn'),
            currentPage: document.getElementById('current-page'),
            searchInput: document.getElementById('search-input'),
            historyTab: document.getElementById('history-tab'),
            favTab: document.getElementById('fav-tab'),
            historyBox: document.getElementById('history-box'),
            favBox: document.getElementById('fav-box'),
            historyCount: document.getElementById('history-count'),
            favCount: document.getElementById('fav-count'),
            clearHistory: document.getElementById('clear-history'),
            addFav: document.getElementById('add-fav'),
            batchOp: document.getElementById('batch-op'),
            loading: document.getElementById('loading'),
            error: document.getElementById('error'),
            errorTxt: document.getElementById('error-txt'),
            retryBtn: document.getElementById('retry-btn'),
            initialTip: document.getElementById('initial-tip'),
            themeToggle: document.getElementById('theme-toggle'),
            helpBtn: document.getElementById('help-btn'),
            // 全屏悬浮控制台元素
            floatCtrl: document.getElementById('float-ctrl'),
            floatCtrlContent: document.getElementById('float-ctrl-content'),
            floatCtrlBtn: document.getElementById('float-ctrl-btn'),
            floatWebsiteUrl: document.getElementById('float-website-url'),
            floatGoBtn: document.getElementById('float-go-btn'),
            floatRefreshBtn: document.getElementById('float-refresh-btn'),
            floatBackBtn: document.getElementById('float-back-btn'),
            floatForwardBtn: document.getElementById('float-forward-btn'),
            floatExitFullBtn: document.getElementById('float-exit-full'),
            floatQrBtn: document.getElementById('float-qr-btn'),
            floatSearchInput: document.getElementById('float-search-input'),
            floatHistoryTab: document.getElementById('float-history-tab'),
            floatFavTab: document.getElementById('float-fav-tab'),
            floatHistoryBox: document.getElementById('float-history-box'),
            floatFavBox: document.getElementById('float-fav-box'),
            floatHistoryCount: document.getElementById('float-history-count'),
            floatFavCount: document.getElementById('float-fav-count'),
            floatThemeToggle: document.getElementById('float-theme-toggle'),
            floatHelpBtn: document.getElementById('float-help-btn'),
            // 弹窗元素
            editHistoryModal: document.getElementById('edit-history-modal'),
            closeEditHistory: document.getElementById('close-edit-history'),
            editHistoryName: document.getElementById('edit-history-name'),
            editHistoryId: document.getElementById('edit-history-id'),
            editHistoryCancel: document.getElementById('edit-history-cancel'),
            editHistorySave: document.getElementById('edit-history-save'),
            favModal: document.getElementById('fav-modal'),
            closeFav: document.getElementById('close-fav'),
            favModalTitle: document.getElementById('fav-modal-title'),
            favName: document.getElementById('fav-name'),
            favUrl: document.getElementById('fav-url'),
            favId: document.getElementById('fav-id'),
            favCancel: document.getElementById('fav-cancel'),
            favSave: document.getElementById('fav-save'),
            qrModal: document.getElementById('qr-modal'),
            closeQr: document.getElementById('close-qr'),
            qrImg: document.getElementById('qr-img'),
            qrTitle: document.getElementById('qr-title'),
            qrMinus: document.getElementById('qr-minus'),
            qrPlus: document.getElementById('qr-plus'),
            qrDownload: document.getElementById('qr-download'),
            qrWrap: document.getElementById('qr-wrap'),
            helpModal: document.getElementById('help-modal'),
            closeHelp: document.getElementById('close-help')
        };

        const CONFIG = {
            STORAGE: {
                HISTORY: 'web_embed_history',
                FAV: 'web_embed_favorite',
                DARK_MODE: 'web_embed_dark',
                LAST_URL: 'web_embed_last_url'
            },
            LAYOUT: {
                HEIGHT_RATIO: 0.88,
                MIN_HEIGHT: 500,
                MAX_WIDTH: 1200
            },
            QR: {
                INIT_SCALE: 1,
                MIN_SCALE: 0.5,
                MAX_SCALE: 2,
                SCALE_STEP: 0.1
            },
            MAX_HISTORY: 50
        };

        let appState = {
            currentUrl: '',
            historyStack: [],
            currentIndex: -1,
            isFullscreen: false,
            qrScale: CONFIG.QR.INIT_SCALE,
            floatCtrlExpanded: false
        };

        // ========== 页面初始化 ==========
        document.addEventListener('DOMContentLoaded', function() {
            adjustIframeLayout();
            window.addEventListener('resize', adjustIframeLayout);
            initDarkMode();
            loadLocalData();
            renderHistory();
            renderFav();
            initFloatCtrlDrag();
            const lastUrl = localStorage.getItem(CONFIG.STORAGE.LAST_URL);
            if (lastUrl) {
                DOM.websiteUrl.value = lastUrl;
                loadWebsite(lastUrl);
            }
            bindAllEvents();
        });

        // ========== 核心功能：布局适配 ==========
        function adjustIframeLayout() {
            if (appState.isFullscreen) return;
            const winWidth = window.innerWidth;
            const winHeight = window.innerHeight;
            const containerWidth = Math.min(winWidth - 32, CONFIG.LAYOUT.MAX_WIDTH);
            const containerHeight = Math.max(winHeight * CONFIG.LAYOUT.HEIGHT_RATIO, CONFIG.LAYOUT.MIN_HEIGHT);
            DOM.iframeContainer.style.width = `${containerWidth}px`;
            DOM.iframeContainer.style.height = `${containerHeight}px`;
            DOM.iframeContainer.style.margin = '0 auto';
        }

        // ========== 核心功能：全屏/非全屏切换（修复白屏） ==========
        function enterFullscreen() {
            if (appState.isFullscreen) return;
            appState.isFullscreen = true;
            document.body.classList.add('fullscreen-mode');
            
            // 修复白屏：隐藏非网页内容，保留iframe容器
            DOM.urlSection.classList.add('hidden');
            DOM.searchSection.classList.add('hidden');
            DOM.controlSection.classList.add('hidden');
            DOM.footerSection.classList.add('hidden');
            document.querySelector('header').classList.add('hidden');
            
            // 修改iframe容器样式，铺满屏幕
            DOM.iframeContainer.style.position = 'fixed';
            DOM.iframeContainer.style.top = '0';
            DOM.iframeContainer.style.left = '0';
            DOM.iframeContainer.style.width = '100vw';
            DOM.iframeContainer.style.height = '100vh';
            DOM.iframeContainer.style.borderRadius = '0';
            DOM.iframeContainer.style.boxShadow = 'none';
            DOM.iframeContainer.style.margin = '0';
            
            // 修改iframe样式
            DOM.mainIframe.classList.add('fullscreen-iframe');
            
            // 显示悬浮控制台
            DOM.floatCtrl.classList.remove('hidden');
            
            // 同步数据到悬浮控制台
            DOM.floatWebsiteUrl.value = appState.currentUrl;
            updateNavBtnState();
            DOM.floatHistoryCount.textContent = DOM.historyCount.textContent;
            DOM.floatFavCount.textContent = DOM.favCount.textContent;
            renderFloatHistory();
            renderFloatFav();
        }

        function exitFullscreen() {
            if (!appState.isFullscreen) return;
            appState.isFullscreen = false;
            document.body.classList.remove('fullscreen-mode');
            
            // 显示所有非网页内容
            DOM.urlSection.classList.remove('hidden');
            DOM.searchSection.classList.remove('hidden');
            DOM.controlSection.classList.remove('hidden');
            DOM.footerSection.classList.remove('hidden');
            document.querySelector('header').classList.remove('hidden');
            
            // 恢复iframe容器样式
            DOM.iframeContainer.style.position = 'relative';
            DOM.iframeContainer.style.top = 'auto';
            DOM.iframeContainer.style.left = 'auto';
            DOM.iframeContainer.style.width = 'auto';
            DOM.iframeContainer.style.height = 'auto';
            DOM.iframeContainer.style.borderRadius = '';
            DOM.iframeContainer.style.boxShadow = '';
            DOM.iframeContainer.style.margin = '0 auto';
            
            // 恢复iframe样式
            DOM.mainIframe.classList.remove('fullscreen-iframe');
            
            // 隐藏悬浮控制台并回缩
            DOM.floatCtrl.classList.add('hidden');
            appState.floatCtrlExpanded = false;
            DOM.floatCtrlContent.classList.remove('expand');
            DOM.floatCtrlBtn.innerHTML = '<i class="fas fa-cog text-2xl"></i>';
            
            // 重新适配非全屏布局
            adjustIframeLayout();
        }

        function toggleFloatCtrl() {
            appState.floatCtrlExpanded = !appState.floatCtrlExpanded;
            if (appState.floatCtrlExpanded) {
                DOM.floatCtrlContent.classList.add('expand');
                DOM.floatCtrlBtn.innerHTML = '<i class="fas fa-times text-2xl"></i>';
            } else {
                DOM.floatCtrlContent.classList.remove('expand');
                DOM.floatCtrlBtn.innerHTML = '<i class="fas fa-cog text-2xl"></i>';
            }
        }

        function initFloatCtrlDrag() {
            const btn = DOM.floatCtrlBtn;
            const wrapper = DOM.floatCtrl;
            let isDragging = false;
            let startX, startY, wrapperX, wrapperY;

            btn.addEventListener('mousedown', (e) => {
                isDragging = true;
                startX = e.clientX;
                startY = e.clientY;
                const wrapperRect = wrapper.getBoundingClientRect();
                wrapperX = wrapperRect.left;
                wrapperY = wrapperRect.top;
                btn.style.cursor = 'grabbing';
                e.stopPropagation();
            });

            document.addEventListener('mousemove', (e) => {
                if (!isDragging) return;
                const dx = e.clientX - startX;
                const dy = e.clientY - startY;
                let newX = wrapperX + dx;
                let newY = wrapperY + dy;
                const winWidth = window.innerWidth;
                const winHeight = window.innerHeight;
                const wrapperWidth = wrapper.offsetWidth;
                const wrapperHeight = wrapper.offsetHeight;
                newX = Math.max(0, Math.min(newX, winWidth - wrapperWidth));
                newY = Math.max(0, Math.min(newY, winHeight - wrapperHeight));
                wrapper.style.left = `${newX}px`;
                wrapper.style.top = `${newY}px`;
                wrapper.style.right = 'auto';
                wrapper.style.bottom = 'auto';
            });

            document.addEventListener('mouseup', () => {
                if (isDragging) {
                    isDragging = false;
                    btn.style.cursor = 'move';
                }
            });

            btn.addEventListener('click', toggleFloatCtrl);
        }

        // ========== 核心功能：网站加载/导航 ==========
        function loadWebsite(url) {
            if (!url.trim()) {
                resetToInitial();
                return;
            }
            if (!url.startsWith('http://') && !url.startsWith('https://')) {
                url = `https://${url}`;
            }
            try {
                new URL(url);
            } catch (e) {
                showError('请输入有效的网站URL（例如：https://www.baidu.com）');
                return;
            }
            showLoading();
            DOM.initialTip.classList.add('hidden');
            appState.currentUrl = url;
            DOM.currentPage.textContent = url;
            DOM.websiteUrl.value = url;
            DOM.floatWebsiteUrl.value = url;
            updateHistoryStack(url);
            DOM.mainIframe.src = url;
            localStorage.setItem(CONFIG.STORAGE.LAST_URL, url);
            DOM.batchOp.classList.remove('hidden');
        }

        function resetToInitial() {
            appState.currentUrl = '';
            appState.historyStack = [];
            appState.currentIndex = -1;
            DOM.currentPage.textContent = '未选择网站';
            DOM.websiteUrl.value = '';
            DOM.floatWebsiteUrl.value = '';
            DOM.mainIframe.src = 'about:blank';
            DOM.initialTip.classList.remove('hidden');
            DOM.batchOp.classList.add('hidden');
            updateNavBtnState();
            localStorage.removeItem(CONFIG.STORAGE.LAST_URL);
            hideLoading();
            hideError();
        }

        function updateHistoryStack(url) {
            if (appState.historyStack[appState.currentIndex] === url) return;
            if (appState.currentIndex < appState.historyStack.length - 1) {
                appState.historyStack = appState.historyStack.slice(0, appState.currentIndex + 1);
            }
            appState.historyStack.push(url);
            appState.currentIndex = appState.historyStack.length - 1;
            updateNavBtnState();
            addToLocalHistory(url, DOM.mainIframe.contentDocument?.title || '未知网站');
        }

        function goBack() {
            if (appState.currentIndex <= 0) return;
            appState.currentIndex--;
            const url = appState.historyStack[appState.currentIndex];
            loadWebsite(url);
        }

        function goForward() {
            if (appState.currentIndex >= appState.historyStack.length - 1) return;
            appState.currentIndex++;
            const url = appState.historyStack[appState.currentIndex];
            loadWebsite(url);
        }

        function refreshWebsite() {
            if (!appState.currentUrl) return;
            showLoading();
            DOM.mainIframe.src = appState.currentUrl;
        }

        function updateNavBtnState() {
            const canBack = appState.currentIndex > 0;
            const canForward = appState.currentIndex < appState.historyStack.length - 1;
            DOM.backBtn.disabled = !canBack;
            DOM.forwardBtn.disabled = !canForward;
            DOM.floatBackBtn.disabled = !canBack;
            DOM.floatForwardBtn.disabled = !canForward;
        }

        // ========== 核心功能：历史记录 ==========
        function loadLocalHistory() {
            const str = localStorage.getItem(CONFIG.STORAGE.HISTORY) || '[]';
            return JSON.parse(str);
        }

        function addToLocalHistory(url, title) {
            let history = loadLocalHistory();
            history = history.filter(item => item.url !== url);
            const shortTitle = title.length > 25 ? `${title.substring(0, 25)}...` : title;
            history.unshift({
                id: Date.now().toString(),
                title: shortTitle,
                url: url,
                time: new Date().toLocaleString()
            });
            if (history.length > CONFIG.MAX_HISTORY) {
                history = history.slice(0, CONFIG.MAX_HISTORY);
            }
            localStorage.setItem(CONFIG.STORAGE.HISTORY, JSON.stringify(history));
            renderHistory();
            renderFloatHistory();
        }

        function renderHistory(keyword = '') {
            let history = loadLocalHistory();
            if (keyword) {
                history = history.filter(item => item.title.includes(keyword) || item.url.includes(keyword));
            }
            DOM.historyCount.textContent = loadLocalHistory().length;
            if (history.length === 0) {
                DOM.historyBox.innerHTML = '<div class="text-center text-gray-500 py-4">暂无历史记录</div>';
                return;
            }
            DOM.historyBox.innerHTML = history.map(item => `
                <div class="flex justify-between items-center p-2 border-b border-gray-100 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-800 rounded-lg">
                    <div class="flex-1 min-w-0">
                        <p class="font-medium text-gray-900 dark:text-white truncate">${item.title}</p>
                        <p class="text-xs text-gray-500 dark:text-gray-400 truncate">${item.url}</p>
                        <p class="text-xs text-gray-400 dark:text-gray-500 mt-1">${item.time}</p>
                    </div>
                    <div class="flex space-x-2 ml-2 shrink-0">
                        <button class="text-blue-500 hover:text-blue-600" onclick="loadWebsite('${item.url}')" title="访问">
                            <i class="fas fa-external-link-alt"></i>
                        </button>
                        <button class="text-yellow-500 hover:text-yellow-600" onclick="openEditHistory('${item.id}')" title="编辑名称">
                            <i class="fas fa-edit"></i>
                        </button>
                        <button class="text-danger hover:text-danger/80" onclick="deleteLocalHistory('${item.id}')" title="删除">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                </div>
            `).join('');
        }

        function renderFloatHistory(keyword = '') {
            let history = loadLocalHistory();
            if (keyword) {
                history = history.filter(item => item.title.includes(keyword) || item.url.includes(keyword));
            }
            DOM.floatHistoryCount.textContent = loadLocalHistory().length;
            if (history.length === 0) {
                DOM.floatHistoryBox.innerHTML = '<div class="text-center text-gray-500 py-2 text-sm">暂无历史记录</div>';
                return;
            }
            DOM.floatHistoryBox.innerHTML = history.map(item => `
                <div class="flex justify-between items-center p-2 border-b border-gray-200 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-800 rounded-lg text-sm">
                    <div class="flex-1 min-w-0 truncate">
                        <p class="font-medium text-gray-900 dark:text-white truncate">${item.title}</p>
                        <p class="text-xs text-gray-500 dark:text-gray-400 truncate">${item.url}</p>
                    </div>
                    <div class="flex space-x-2 ml-2 shrink-0">
                        <button class="text-blue-500 hover:text-blue-600" onclick="loadWebsite('${item.url}')" title="访问">
                            <i class="fas fa-external-link-alt"></i>
                        </button>
                        <button class="text-danger hover:text-danger/80" onclick="deleteLocalHistory('${item.id}')" title="删除">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                </div>
            `).join('');
        }

        function openEditHistory(id) {
            const history = loadLocalHistory();
            const item = history.find(item => item.id === id);
            if (!item) return;
            DOM.editHistoryId.value = id;
            DOM.editHistoryName.value = item.title;
            DOM.editHistoryModal.classList.remove('hidden');
        }

        function saveEditHistory() {
            const id = DOM.editHistoryId.value;
            const newTitle = DOM.editHistoryName.value.trim();
            if (!newTitle) {
                alert('网页名称不能为空！');
                return;
            }
            let history = loadLocalHistory();
            const index = history.findIndex(item => item.id === id);
            if (index === -1) return;
            history[index].title = newTitle;
            localStorage.setItem(CONFIG.STORAGE.HISTORY, JSON.stringify(history));
            renderHistory(DOM.searchInput.value.trim());
            renderFloatHistory(DOM.floatSearchInput.value.trim());
            DOM.editHistoryModal.classList.add('hidden');
        }

        function deleteLocalHistory(id) {
            let history = loadLocalHistory();
            history = history.filter(item => item.id !== id);
            localStorage.setItem(CONFIG.STORAGE.HISTORY, JSON.stringify(history));
            renderHistory(DOM.searchInput.value.trim());
            renderFloatHistory(DOM.floatSearchInput.value.trim());
        }

        function clearLocalHistory() {
            if (confirm('确定要清空所有历史记录吗？此操作不可恢复！')) {
                localStorage.removeItem(CONFIG.STORAGE.HISTORY);
                renderHistory();
                renderFloatHistory();
            }
        }

        // ========== 核心功能：收藏夹 ==========
        function loadLocalFav() {
            const str = localStorage.getItem(CONFIG.STORAGE.FAV) || '[]';
            return JSON.parse(str);
        }

        function saveLocalFav() {
            const id = DOM.favId.value;
            const title = DOM.favName.value.trim();
            const url = DOM.favUrl.value.trim();
            if (!title || !url) {
                alert('收藏名称和URL不能为空！');
                return;
            }
            try {
                new URL(url);
            } catch (e) {
                alert('请输入有效的网站URL！');
                return;
            }
            let fav = loadLocalFav();
            if (id) {
                const index = fav.findIndex(item => item.id === id);
                if (index !== -1) {
                    fav[index].title = title;
                    fav[index].url = url;
                }
            } else {
                if (fav.some(item => item.url === url)) {
                    alert('该网站已在收藏夹中！');
                    return;
                }
                fav.unshift({
                    id: Date.now().toString(),
                    title: title,
                    url: url
                });
            }
            localStorage.setItem(CONFIG.STORAGE.FAV, JSON.stringify(fav));
            renderFav(DOM.searchInput.value.trim());
            renderFloatFav(DOM.floatSearchInput.value.trim());
            DOM.favModal.classList.add('hidden');
        }

        function openFavModal(id = '') {
            if (id) {
                const fav = loadLocalFav();
                const item = fav.find(item => item.id === id);
                if (!item) return;
                DOM.favModalTitle.textContent = '编辑收藏';
                DOM.favName.value = item.title;
                DOM.favUrl.value = item.url;
                DOM.favId.value = id;
            } else {
                DOM.favModalTitle.textContent = '添加收藏';
                DOM.favName.value = DOM.mainIframe.contentDocument?.title || '未知网站';
                DOM.favUrl.value = appState.currentUrl;
                DOM.favId.value = '';
            }
            DOM.favModal.classList.remove('hidden');
        }

        function deleteLocalFav(id) {
            let fav = loadLocalFav();
            fav = fav.filter(item => item.id !== id);
            localStorage.setItem(CONFIG.STORAGE.FAV, JSON.stringify(fav));
            renderFav(DOM.searchInput.value.trim());
            renderFloatFav(DOM.floatSearchInput.value.trim());
        }

        function renderFav(keyword = '') {
            let fav = loadLocalFav();
            if (keyword) {
                fav = fav.filter(item => item.title.includes(keyword) || item.url.includes(keyword));
            }
            DOM.favCount.textContent = loadLocalFav().length;
            if (fav.length === 0) {
                DOM.favBox.innerHTML = '<div class="text-center text-gray-500 py-4">暂无收藏</div>';
                return;
            }
            DOM.favBox.innerHTML = fav.map(item => `
                <div class="flex justify-between items-center p-2 border-b border-gray-100 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-800 rounded-lg">
                    <div class="flex-1 min-w-0">
                        <p class="font-medium text-gray-900 dark:text-white truncate">${item.title}</p>
                        <p class="text-xs text-gray-500 dark:text-gray-400 truncate">${item.url}</p>
                    </div>
                    <div class="flex space-x-2 ml-2 shrink-0">
                        <button class="text-blue-500 hover:text-blue-600" onclick="loadWebsite('${item.url}')" title="访问">
                            <i class="fas fa-external-link-alt"></i>
                        </button>
                        <button class="text-yellow-500 hover:text-yellow-600" onclick="openFavModal('${item.id}')" title="编辑">
                            <i class="fas fa-edit"></i>
                        </button>
                        <button class="text-danger hover:text-danger/80" onclick="deleteLocalFav('${item.id}')" title="删除">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                </div>
            `).join('');
        }

        function renderFloatFav(keyword = '') {
            let fav = loadLocalFav();
            if (keyword) {
                fav = fav.filter(item => item.title.includes(keyword) || item.url.includes(keyword));
            }
            DOM.floatFavCount.textContent = loadLocalFav().length;
            if (fav.length === 0) {
                DOM.floatFavBox.innerHTML = '<div class="text-center text-gray-500 py-2 text-sm">暂无收藏</div>';
                return;
            }
            DOM.floatFavBox.innerHTML = fav.map(item => `
                <div class="flex justify-between items-center p-2 border-b border-gray-200 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-800 rounded-lg text-sm">
                    <div class="flex-1 min-w-0 truncate">
                        <p class="font-medium text-gray-900 dark:text-white truncate">${item.title}</p>
                        <p class="text-xs text-gray-500 dark:text-gray-400 truncate">${item.url}</p>
                    </div>
                    <div class="flex space-x-2 ml-2 shrink-0">
                        <button class="text-blue-500 hover:text-blue-600" onclick="loadWebsite('${item.url}')" title="访问">
                            <i class="fas fa-external-link-alt"></i>
                        </button>
                        <button class="text-danger hover:text-danger/80" onclick="deleteLocalFav('${item.id}')" title="删除">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                </div>
            `).join('');
        }

        // ========== 核心功能：二维码生成 ==========
        function generateQrCode() {
            if (!appState.currentUrl) {
                alert('请先加载一个网站再生成二维码！');
                return;
            }
            DOM.qrTitle.textContent = DOM.mainIframe.contentDocument?.title || appState.currentUrl;
            DOM.qrImg.innerHTML = '';
            appState.qrScale = CONFIG.QR.INIT_SCALE;
            QRCode.toCanvas(DOM.qrImg, appState.currentUrl, {
                width: 200,
                margin: 1,
                color: {
                    dark: document.body.classList.contains('dark') ? '#f3f4f6' : '#000000',
                    light: document.body.classList.contains('dark') ? '#1f2937' : '#ffffff'
                }
            }, function (error) {
                if (error) {
                    console.error(error);
                    alert('二维码生成失败，请重试！');
                } else {
                    DOM.qrModal.classList.remove('hidden');
                }
            });
        }

        function scaleQrCode(step) {
            appState.qrScale = Math.max(CONFIG.QR.MIN_SCALE, Math.min(CONFIG.QR.MAX_SCALE, appState.qrScale + step));
            const canvas = DOM.qrImg.querySelector('canvas');
            if (canvas) {
                canvas.style.transform = `scale(${appState.qrScale})`;
                canvas.style.transformOrigin = 'center';
            }
        }

        function downloadQrCode() {
            html2canvas(DOM.qrWrap, {
                scale: 2,
                backgroundColor: document.body.classList.contains('dark') ? '#1f2937' : '#ffffff'
            }).then(canvas => {
                const link = document.createElement('a');
                link.download = `${DOM.qrTitle.textContent.replace(/[\\/:*?"<>|]/g, '')}_二维码.png`;
                link.href = canvas.toDataURL();
                link.click();
            }).catch(err => {
                console.error(err);
                alert('下载失败，请重试！');
            });
        }

        // ========== 辅助功能：暗色模式 ==========
        function initDarkMode() {
            const isDark = localStorage.getItem(CONFIG.STORAGE.DARK_MODE) === 'true' || 
                          (!localStorage.getItem(CONFIG.STORAGE.DARK_MODE) && 
                          window.matchMedia('(prefers-color-scheme: dark)').matches);
            if (isDark) {
                document.body.classList.add('dark');
                DOM.themeToggle.innerHTML = '<i class="fas fa-sun text-yellow-400"></i>';
                DOM.floatThemeToggle.innerHTML = '<i class="fas fa-sun text-yellow-400"></i>';
            } else {
                document.body.classList.remove('dark');
                DOM.themeToggle.innerHTML = '<i class="fas fa-moon text-gray-600"></i>';
                DOM.floatThemeToggle.innerHTML = '<i class="fas fa-moon text-gray-600"></i>';
            }
        }

        function toggleDarkMode() {
            document.body.classList.toggle('dark');
            const isDark = document.body.classList.contains('dark');
            localStorage.setItem(CONFIG.STORAGE.DARK_MODE, isDark);
            if (isDark) {
                DOM.themeToggle.innerHTML = '<i class="fas fa-sun text-yellow-400"></i>';
                DOM.floatThemeToggle.innerHTML = '<i class="fas fa-sun text-yellow-400"></i>';
            } else {
                DOM.themeToggle.innerHTML = '<i class="fas fa-moon text-gray-600"></i>';
                DOM.floatThemeToggle.innerHTML = '<i class="fas fa-moon text-gray-600"></i>';
            }
        }

        // ========== 辅助功能：加载/错误提示 ==========
        function showLoading() {
            DOM.loading.classList.remove('hidden');
            hideError();
        }

        function hideLoading() {
            DOM.loading.classList.add('hidden');
        }

        function showError(message) {
            DOM.errorTxt.textContent = message || '无法加载指定的网站，请检查URL是否正确或尝试其他网站。';
            DOM.error.classList.remove('hidden');
            hideLoading();
        }

        function hideError() {
            DOM.error.classList.add('hidden');
        }

        // ========== 辅助功能：本地数据加载 ==========
        function loadLocalData() {
            // 加载暗色模式
            initDarkMode();
            // 监听系统暗色模式变化
            window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', initDarkMode);
        }

        // ========== 事件绑定 ==========
        function bindAllEvents() {
            // 非全屏事件
            DOM.goBtn.addEventListener('click', () => loadWebsite(DOM.websiteUrl.value));
            DOM.refreshBtn.addEventListener('click', refreshWebsite);
            DOM.backBtn.addEventListener('click', goBack);
            DOM.forwardBtn.addEventListener('click', goForward);
            DOM.enterFullBtn.addEventListener('click', enterFullscreen);
            DOM.qrBtn.addEventListener('click', generateQrCode);
            DOM.websiteUrl.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') loadWebsite(DOM.websiteUrl.value);
            });
            DOM.searchInput.addEventListener('input', (e) => {
                renderHistory(e.target.value.trim());
                renderFloatHistory(e.target.value.trim());
            });
            DOM.historyTab.addEventListener('click', () => {
                DOM.historyTab.classList.add('tab-active');
                DOM.favTab.classList.remove('tab-active');
                DOM.historyBox.classList.remove('hidden');
                DOM.favBox.classList.add('hidden');
            });
            DOM.favTab.addEventListener('click', () => {
                DOM.favTab.classList.add('tab-active');
                DOM.historyTab.classList.remove('tab-active');
                DOM.favBox.classList.remove('hidden');
                DOM.historyBox.classList.add('hidden');
            });
            DOM.clearHistory.addEventListener('click', clearLocalHistory);
            DOM.addFav.addEventListener('click', () => openFavModal());
            DOM.retryBtn.addEventListener('click', () => loadWebsite(appState.currentUrl));
            DOM.themeToggle.addEventListener('click', toggleDarkMode);
            DOM.helpBtn.addEventListener('click', () => DOM.helpModal.classList.remove('hidden'));

            // 全屏悬浮控制台事件
            DOM.floatGoBtn.addEventListener('click', () => loadWebsite(DOM.floatWebsiteUrl.value));
            DOM.floatRefreshBtn.addEventListener('click', refreshWebsite);
            DOM.floatBackBtn.addEventListener('click', goBack);
            DOM.floatForwardBtn.addEventListener('click', goForward);
            DOM.floatExitFullBtn.addEventListener('click', exitFullscreen);
            DOM.floatQrBtn.addEventListener('click', generateQrCode);
            DOM.floatWebsiteUrl.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') loadWebsite(DOM.floatWebsiteUrl.value);
            });
            DOM.floatSearchInput.addEventListener('input', (e) => {
                renderHistory(e.target.value.trim());
                renderFloatHistory(e.target.value.trim());
            });
            DOM.floatHistoryTab.addEventListener('click', () => {
                DOM.floatHistoryTab.classList.add('tab-active');
                DOM.floatFavTab.classList.remove('tab-active');
                DOM.floatHistoryBox.classList.remove('hidden');
                DOM.floatFavBox.classList.add('hidden');
            });
            DOM.floatFavTab.addEventListener('click', () => {
                DOM.floatFavTab.classList.add('tab-active');
                DOM.floatHistoryTab.classList.remove('tab-active');
                DOM.floatFavBox.classList.remove('hidden');
                DOM.floatHistoryBox.classList.add('hidden');
            });
            DOM.floatThemeToggle.addEventListener('click', toggleDarkMode);
            DOM.floatHelpBtn.addEventListener('click', () => DOM.helpModal.classList.remove('hidden'));

            // 弹窗事件
            // 历史编辑弹窗
            DOM.closeEditHistory.addEventListener('click', () => DOM.editHistoryModal.classList.add('hidden'));
            DOM.editHistoryCancel.addEventListener('click', () => DOM.editHistoryModal.classList.add('hidden'));
            DOM.editHistorySave.addEventListener('click', saveEditHistory);
            // 收藏弹窗
            DOM.closeFav.addEventListener('click', () => DOM.favModal.classList.add('hidden'));
            DOM.favCancel.addEventListener('click', () => DOM.favModal.classList.add('hidden'));
            DOM.favSave.addEventListener('click', saveLocalFav);
            // 二维码弹窗
            DOM.closeQr.addEventListener('click', () => DOM.qrModal.classList.add('hidden'));
            DOM.qrMinus.addEventListener('click', () => scaleQrCode(-CONFIG.QR.SCALE_STEP));
            DOM.qrPlus.addEventListener('click', () => scaleQrCode(CONFIG.QR.SCALE_STEP));
            DOM.qrDownload.addEventListener('click', downloadQrCode);
            // 帮助弹窗
            DOM.closeHelp.addEventListener('click', () => DOM.helpModal.classList.add('hidden'));

            // iframe事件
            DOM.mainIframe.addEventListener('load', () => {
                hideLoading();
                if (DOM.mainIframe.contentDocument?.title) {
                    DOM.currentPage.textContent = DOM.mainIframe.contentDocument.title;
                    // 更新历史记录的标题
                    const lastHistory = loadLocalHistory()[0];
                    if (lastHistory && lastHistory.url === appState.currentUrl) {
                        let history = loadLocalHistory();
                        history[0].title = DOM.mainIframe.contentDocument.title.length > 25 
                            ? `${DOM.mainIframe.contentDocument.title.substring(0, 25)}...` 
                            : DOM.mainIframe.contentDocument.title;
                        localStorage.setItem(CONFIG.STORAGE.HISTORY, JSON.stringify(history));
                        renderHistory(DOM.searchInput.value.trim());
                        renderFloatHistory(DOM.floatSearchInput.value.trim());
                    }
                }
            });
            DOM.mainIframe.addEventListener('error', () => {
                showError('无法加载指定的网站，可能是网站不允许嵌入或网络问题。');
            });

            // 全屏事件监听
            document.addEventListener('fullscreenchange', () => {
                if (!document.fullscreenElement && appState.isFullscreen) {
                    exitFullscreen();
                }
            });
            document.addEventListener('webkitfullscreenchange', () => {
                if (!document.webkitFullscreenElement && appState.isFullscreen) {
                    exitFullscreen();
                }
            });
            document.addEventListener('mozfullscreenchange', () => {
                if (!document.mozFullScreenElement && appState.isFullscreen) {
                    exitFullscreen();
                }
            });

            // 点击弹窗外部关闭弹窗
            document.addEventListener('click', (e) => {
                if (e.target.classList.contains('modal')) {
                    DOM.editHistoryModal.classList.add('hidden');
                    DOM.favModal.classList.add('hidden');
                    DOM.qrModal.classList.add('hidden');
                    DOM.helpModal.classList.add('hidden');
                }
            });
        }
    </script>
</body>
</html>
