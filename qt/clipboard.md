# Qt 剪切板

作者：康林<kl222@126.com>

---------------------------

以下 Qt 源代码版本是：Qt5.12.12

### 剪切板接口类关系：

- 外部接口
  - 类 QClipboard 是剪切板的 API
  - 类 QMimeData 是剪切板数据的 API 
- Qt 内部接口
  - 类 QPlatformClipboard 是 Qt 内部的剪切板接口
  - 类 QInternalMimeData 是 Qt 内部的剪切板数据的接口
 
```mermaid

classDiagram
    QGuiApplication *--> QClipboard: clipboard()
    QClipboard --> QGuiApplicationPrivate
    QClipboard o--> QMimeData

    QGuiApplication "1" *--> "1" QGuiApplicationPrivate
    QGuiApplicationPrivate *--> QPlatformIntegration: platformIntegration()
    QPlatformIntegration *--> QPlatformClipboard: clipboard()
    QPlatformClipboard o--> QMimeData

    QMimeData <|-- QInternalMimeData
    
    QGuiApplication: + clipboard()$ QClipboard *
    class QClipboard {
        + mimeData(Mode mode = Clipboard ) const QMimeData *
        + setMimeData(QMimeData *data, Mode mode = Clipboard) void
    }

    class QGuiApplicationPrivate {
        + platformIntegration() QPlatformIntegration*
    }

    <<Interface>> QPlatformIntegration
    QPlatformIntegration: + clipboard() QPlatformClipboard*

    <<Interface>> QPlatformClipboard
    class QPlatformClipboard {
        + mimeData(Mode mode = Clipboard ) const QMimeData *
        + setMimeData(QMimeData *data, Mode mode = Clipboard) void
    }

    <<Interface>> QMimeData
    class QMimeData {
        + formats() QStringList
	+ hasFormat(const QString &mimetype) bool
        # retrieveData(const QString &mimetype, QVariant::Type preferredType) virtual QVariant
    }
    
    <<Interface>> QInternalMimeData
    class QInternalMimeData {
        + formatsHelper(const QMimeData *data)$ QStringList 
        + hasFormatHelper(const QString &mimeType, const QMimeData *data)$ bool
        + renderDataHelper(const QString &mimeType, const QMimeData *data)$ QByteArray
        # hasFormat_sys(const QString &mimeType) bool
        # formats_sys() QStringList
        # retrieveData_sys(const QString &mimeType, QVariant::Type type) QVariant
    }

```

### 各平台接口类：

```mermaid

classDiagram
    QGuiApplication *--> QClipboard: clipboard()
    QClipboard --> QGuiApplicationPrivate
    QClipboard o--> QMimeData

    QGuiApplication "1" *--> "1" QGuiApplicationPrivate
    QGuiApplicationPrivate *--> QPlatformIntegration: platformIntegration()
    QPlatformClipboard o--> QMimeData
    QMimeData <|-- QInternalMimeData

    QPlatformIntegration *--> QPlatformClipboard: clipboard()
    
    QPlatformIntegration <|-- QWindowsIntegration
    QPlatformIntegration <|-- QXcbIntegration
    QPlatformIntegration <|-- QIOSIntegration
    QPlatformIntegration <|-- QAndroidPlatformIntegration

    QPlatformClipboard <|-- QWindowsClipboard
    QPlatformClipboard <|-- QXcbClipboard
    QPlatformClipboard <|-- QIOSClipboard
    QPlatformClipboard <|-- QAndroidPlatformClipboard
    QPlatformClipboard <|-- QQnxClipboard
    QPlatformClipboard <|-- QWinRTClipboard

    QInternalMimeData <|-- QWindowsInternalMimeData
    QInternalMimeData <|-- QXcbMime
    QMimeData <|-- QIOSMimeData

    QGuiApplication: + clipboard()$ QClipboard *
    class QClipboard {
        + mimeData(Mode mode = Clipboard ) const QMimeData *
        + setMimeData(QMimeData *data, Mode mode = Clipboard) void
    }

    class QGuiApplicationPrivate {
        + platformIntegration() QPlatformIntegration*
    }

    <<Interface>> QMimeData
    class QMimeData {
        + formats() QStringList
	+ hasFormat(const QString &mimetype) bool
        # retrieveData(const QString &mimetype, QVariant::Type preferredType) virtual QVariant
    }
    
    <<Interface>> QInternalMimeData
    class QInternalMimeData {
        + formatsHelper(const QMimeData *data)$ QStringList 
        + hasFormatHelper(const QString &mimeType, const QMimeData *data)$ bool
        + renderDataHelper(const QString &mimeType, const QMimeData *data)$ QByteArray
        # hasFormat_sys(const QString &mimeType) bool
        # formats_sys() QStringList
        # retrieveData_sys(const QString &mimeType, QVariant::Type type) QVariant
    }

    <<Interface>> QPlatformClipboard
    class QPlatformClipboard {
        + mimeData(Mode mode = Clipboard ) const QMimeData *
        + setMimeData(QMimeData *data, Mode mode = Clipboard) void
    }

    <<Interface>> QPlatformIntegration
    QPlatformIntegration: + clipboard() QPlatformClipboard*

```


### 剪切板 Windows 平台类关系：

- QWindowsMimeText: 处理文本格式("text/plain"、CF_UNICODETEXT、CF_TEXT)
- QWindowsMimeHtml: 处理 "HTML Format" 格式
- QWindowsMimeURI：处理URL("CF_HDROP"、"UniformResourceLocatorW"、"UniformResourceLocator")
- QWindowsMimeImage: 处理图片(CF_DIB、CF_DIBV5、"PNG")
- QBuiltInMimes：处理Qt内部格式（"application/x-color"）
- **QLastResortMimes**：处理除了上面特定格式外的格式。 根据 QMimeData 中的格式在剪切板中注册相应的格式。
例如："FileGroupDescriptorW"（CFSTR_FILEDESCRIPTOR）、”FileContents"（CFSTR_FILECONTENTS）

```mermaid

classDiagram
    QGuiApplication *--> QClipboard: clipboard()
    QClipboard --> QGuiApplicationPrivate
    QClipboard o--> QMimeData

    QGuiApplication "1" *--> "1" QGuiApplicationPrivate
    QGuiApplicationPrivate *--> QPlatformIntegration: platformIntegration()
    QPlatformIntegration *--> QPlatformClipboard: clipboard()
    QPlatformClipboard o--> QMimeData
    QMimeData <|-- QInternalMimeData
    QInternalMimeData <|-- QWindowsInternalMimeData
    QWindowsInternalMimeData <|-- QWindowsClipboardRetrievalMimeData
    
    QPlatformIntegration <|-- QWindowsIntegration
    QPlatformClipboard <|-- QWindowsClipboard

    QWindowsIntegration *--> QWindowsClipboard: clipboard()
    IDataObject <|-- QWindowsOleDataObject

    QWindowsClipboard *--> QWindowsClipboardRetrievalMimeData: m_retrievalData
    QWindowsClipboard *--> QWindowsOleDataObject: m_data
    
    QWindowsOleDataObject --> QWindowsMimeConverter
    QWindowsOleDataObject o--> QMimeData

    QWindowsContext *--> QWindowsMimeConverter
    QWindowsMimeConverter "1" *--> "*" QWindowsMime: m_mimes

    QWindowsMime <|-- QWindowsMimeText
    QWindowsMime <|-- QWindowsMimeHtml
    QWindowsMime <|-- QWindowsMimeURI
    QWindowsMime <|-- QWindowsMimeImage
    QWindowsMime <|-- QLastResortMimes
    QWindowsMime <|-- QBuiltInMimes
    
    QGuiApplication: + clipboard()$ QClipboard *
    class QClipboard {
        + mimeData(Mode mode = Clipboard ) const QMimeData *
        + setMimeData(QMimeData *data, Mode mode = Clipboard) void
    }
    
    <<Interface>> QMimeData
    class QMimeData {
        + formats() QStringList
	+ hasFormat(const QString &mimetype) bool
        # retrieveData(const QString &mimetype, QVariant::Type preferredType) virtual QVariant
    }
    
    <<Interface>> QInternalMimeData
    class QInternalMimeData {
        + formatsHelper(const QMimeData *data)$ QStringList 
        + hasFormatHelper(const QString &mimeType, const QMimeData *data)$ bool
        + renderDataHelper(const QString &mimeType, const QMimeData *data)$ QByteArray
        # hasFormat_sys(const QString &mimeType) bool
        # formats_sys() QStringList
        # retrieveData_sys(const QString &mimeType, QVariant::Type type) QVariant
    }
    
    QGuiApplicationPrivate: + platformIntegration() QPlatformIntegration*
    <<Interface>> QPlatformIntegration
    QPlatformIntegration: + clipboard() QPlatformClipboard*
    <<Interface>> QPlatformClipboard
    class QPlatformClipboard {
        + mimeData(Mode mode = Clipboard ) const QMimeData *
        + setMimeData(QMimeData *data, Mode mode = Clipboard) void
    }
    class QWindowsClipboard {
        + QWindowsClipboardRetrievalMimeData m_retrievalData
        + QWindowsOleDataObject *m_data
    }
    
    class QWindowsOleDataObject{
        - QPointer<QMimeData> data
        + STDMETHOD(GetData)(LPFORMATETC pformatetcIn, LPSTGMEDIUM pmedium);
        + STDMETHOD(GetDataHere)(LPFORMATETC pformatetc, LPSTGMEDIUM pmedium);
        + STDMETHOD(QueryGetData)(LPFORMATETC pformatetc);
        + STDMETHOD(GetCanonicalFormatEtc)(LPFORMATETC pformatetc, LPFORMATETC pformatetcOut);
        + STDMETHOD(SetData)(LPFORMATETC pformatetc, STGMEDIUM FAR * pmedium, BOOL fRelease);
        + STDMETHOD(EnumFormatEtc)(DWORD dwDirection, LPENUMFORMATETC FAR* ppenumFormatEtc);
        + STDMETHOD(DAdvise)(FORMATETC FAR* pFormatetc, DWORD advf, LPADVISESINK pAdvSink, DWORD FAR* pdwConnection);
        + STDMETHOD(DUnadvise)(DWORD dwConnection);
        + STDMETHOD(EnumDAdvise)(LPENUMSTATDATA FAR* ppenumAdvise);
    }
    class QWindowsMimeConverter {
        + converterToMime(const FORMATETC &formatetc, const QMimeData *mimeData) QWindowsMime *
        - QList~QWindowsMime *~ m_mimes
    }

    <<Abstract>> QWindowsMime
    class QWindowsMime {
        + canConvertFromMime(const FORMATETC &formatetc, const QMimeData *mimeData)* virtual bool
        + onvertFromMime(const FORMATETC &formatetc, const QMimeData *mimeData, STGMEDIUM * pmedium)* virtual bool 
        + formatsForMime(const QString &mimeType, const QMimeData *mimeData)* virtual QVector~FORMATETC~

        + canConvertToMime(const QString &mimeType, IDataObject *pDataObj)* virtual bool
        + onvertToMime(const QString &mimeType, IDataObject *pDataObj, QVariant::Type preferredType)* virtual QVariant
        + mimeForFormat(const FORMATETC &formatetc)* virtual QString
    }

```

