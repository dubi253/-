# 基于可注入的官方基础镜像（默认保留原有 latest 行为）
ARG BASE_IMAGE=openwebui/open-webui:latest
FROM ${BASE_IMAGE}

# 切换到 root 用户
USER root

# 1. 安装系统级依赖、中文字体及多语言 OCR 包
RUN apt-get update && apt-get install -y --no-install-recommends \
    # 文档解析核心工具
    poppler-utils \
    libmagic1 \
    ffmpeg \
    pandoc \
    # LibreOffice 完整套件（支持旧版 .doc/.ppt/.xls 转换）
    libreoffice \
    # 中文字体（防止 LibreOffice 转换时乱码）
    fonts-wqy-zenhei \
    # Tesseract OCR 引擎及多语言包（简繁中英日韩）
    tesseract-ocr \
    tesseract-ocr-eng \
    tesseract-ocr-chi-sim \
    tesseract-ocr-chi-tra \
    tesseract-ocr-jpn \
    tesseract-ocr-kor \
    # 中文 Locale 支持
    locales \
    && locale-gen zh_CN.UTF-8 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 设置中文环境变量，防止文件名或文本提取乱码
ENV LANG=zh_CN.UTF-8 \
    LANGUAGE=zh_CN:zh \
    LC_ALL=zh_CN.UTF-8

# 2. 使用 uv 包管理器安装 Python 解析依赖
RUN uv pip install --system --no-cache-dir \
    unstructured \
    python-magic \
    pdf2image \
    pytesseract \
    pypdf \
    pdfplumber \
    python-docx \
    python-pptx \
    openpyxl \
    markdown \
    beautifulsoup4 \
    chardet

# 切回默认非 root 用户
USER 1000:1000

# 保持官方启动脚本
CMD ["bash", "start.sh"]
