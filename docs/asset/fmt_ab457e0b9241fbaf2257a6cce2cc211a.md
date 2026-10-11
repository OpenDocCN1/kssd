
# Qt 内部机制及逆向

> 原文：[`www.pediy.com/kssd/pediy12/133181.html`](https://www.pediy.com/kssd/pediy12/133181.html)

【翻译】Qt 内部机制及逆向

原作者：Daniel Pistelli；

翻 译：zouzhin

参加看雪有很长一段时间了，一直无所贡献，真是有愧各位同坛好友。前不久发了个 Qt 求助帖 `http://bbs.pediy.com/showthread.php?t=132491`，没人回复，刚好看到了国外牛人 Daniel Pistelli 写的《Qt Internals & Reversing》，就翻译一下给需要的人做个参考。由于 E 文水平不高，有不对的地方多包涵，高手请直接看原文 `http://ntcore.com/files/qtrev.htm`。

开始之前，简单补充介绍一下 Qt：

用官网的话说：Qt 一个开源跨平台应用程序和 UI 开发框架。优点有：

针对多个平台只需编写一次代码

使用 `Qt` 您只需编写一次应用程序和 `UI`，无须重新编写源代码，便可跨不同的桌面和嵌入式操作系统进行部署，既节省了时间又降低开发成本。

创建令人意想不到的用户体验

`Qt` 提供了应用程序生成块，包括庞大的可定制 `widget` 集合、图形画布、风格引擎和其他内容，您可用来生成新颖的用户界面。由于集成了 3D 图形、多媒体音频或视频、视觉效果、动画和自定义风格，使其在竞争中脱颖而出。

事半功倍(且倍道而进)

无论是使用全新的 `Qt Creator` 跨平台 `IDE` 还是仅是 `Qt` 本身，`Qt` 都易学易用。而且由于有了 `Qt` 模块化的类库，您可以更多地关注创新，无须在平台本身编码上花费过多时间，这样就可将软件快速推向市场。

在单一应用程序中混合网络和本地代码

由于 `Qt` 集成了 `WebKit` 网络渲染引擎，您可以快速地(查看混合方式) 将网络内容和服务集成到本地应用程序中，还可以利用网络环境提供您的服务和功能，让您的用户在使用过程中留下深刻印象。

用本文原作者的话：“in my opinion, the Qt framework will be used more and more by software developers”.个人也觉得 `Qt` 是有其优势，有兴趣的可以利用一下。

以下正式开始

## （一）内部机制

我见过的最严谨的 C++框架就是 `Qt` 框架，`Qt` 将 C++带入了一个新的高度。`Qt` 引入的信号（signal）和槽（slot）技术很有创意，其中一点就是，一个对象可以不要声明就可以调用其它对象的方法。为了运作信号和槽，`Qt` 采用了动态化机制（dynamism）。这种动态化机制可以由 `Qt` 框架自动实现，也可以由开发人员通过 `QMetaObject` 类手动实现。有关信号和槽的内容可以参考 `http://doc.trolltech.com/4.7/signalsandslots.html`。

我们看一个简单的信号和槽的例子：

```cpp
// sas.h

#include <QObject>

class Counter : public QObject
{
  Q_OBJECT
public:
  Counter() { m_value = 0; };
  int value() const { return m_value; };

public slots:
  void setValue(int value)
  {
    if (value != m_value)
    {
      m_value = value;
      emit valueChanged(value);
    }
  };

signals:
  void valueChanged(int newValue);

private:
  int m_value;
};

// main.cpp

#include "sas.h"

int main(int argc, char *argv[])
{
  Counter a, b;
  QObject::connect(&a, SIGNAL(valueChanged(int)),
    &b, SLOT(setValue(int)));
  a.setValue(12);     // a.value() == 12, b.value() == 12
  b.setValue(48);     // a.value() == 12, b.value() == 48
  return 0;
}
```

`SIGNAL` 和 `SLOT` 宏将括号中的内容封装成一个字符串，同时还附加一个 ID 号，如下所示：

```cpp
#define SLOT(a)         "1"#a
#define SIGNAL(a)       "2"#a
```

所以，也可以直接这么写 `connect` 函数：

`QObject::connect(&a, "2valueChanged(int)", &b, "1setValue(int)");`

`signals` 和 `slots` 是 Qt 关键词，可以在头文件中找到，只用于 Qt 的元编译器（`moc`）。

```cpp
# if defined(QT_NO_KEYWORDS)
#  define QT_NO_EMIT
# else
#   define slots
#   define signals protected
# endif

# define Q_SLOTS
# define Q_SIGNALS protected
# define Q_PRIVATE_SLOT(d, signature)
# define Q_EMIT

#ifndef QT_NO_EMIT
# define emit
#endif
```

`emit` 宏没什么需要解释的，`signals` 宏有点不同，它限定 `Qt` 信号为 `protected` 方法，而 `slots` 宏可以是任意类型。

我们接下来看看 `Q_OBJECT` 宏：

```cpp
/* tmake ignore Q_OBJECT */

#define Q_OBJECT_CHECK \
    template <typename T> inline void qt_check_for_QOBJECT_macro(const T &_q_argument) \
const \
    { int i = qYouForgotTheQ_OBJECT_Macro(this, &_q_argument); i = i; }

template <typename T>
inline int qYouForgotTheQ_OBJECT_Macro(T, T) { return 0; }

template <typename T1, typename T2>
inline void qYouForgotTheQ_OBJECT_Macro(T1, T2) {}

#endif // QT_NO_MEMBER_TEMPLATES

/* tmake ignore Q_OBJECT */

#define Q_OBJECT \
public: \
    Q_OBJECT_CHECK \
    static const QMetaObject staticMetaObject; \
    virtual const QMetaObject *metaObject() const; \
    virtual void *qt_metacast(const char *); \
    QT_TR_FUNCTIONS \
    virtual int qt_metacall(QMetaObject::Call, int, void **); \
private:
```

`staticMetaObject` 是 `QMetaObject` 对象，因为需要给属于同一类的全部实例共享，所以它是静态的。`metaObject` 方法仅仅返回 `staticMetaObject`。`QT_TR_FUNCTIONS` 是一个用于所有 `tr` 函数的宏，用来实现多语言支持。`qt_metacast` 用于按照类名或它的某个基类名进行动态转换（`dynamic cast`）【Qt 显然不依赖运行时类型检查（RTTI）】。`qt_metacall` 通过索引调用内部信号和槽。以下是 `QMetaObject` 的声明：

```cpp
struct Q_CORE_EXPORT QMetaObject
{
    const char *className() const;
    const QMetaObject *superClass() const;
    QObject *cast(QObject *obj) const;

#ifndef QT_NO_TRANSLATION
    // ### Qt 4: Merge overloads
    QString tr(const char *s, const char *c) const;
    QString trUtf8(const char *s, const char *c) const;
    QString tr(const char *s, const char *c, int n) const;
    QString trUtf8(const char *s, const char *c, int n) const;
#endif // QT_NO_TRANSLATION

    int methodOffset() const;
    int enumeratorOffset() const;
    int propertyOffset() const;
    int classInfoOffset() const;
    int methodCount() const;
    int enumeratorCount() const;
    int propertyCount() const;
    int classInfoCount() const;
    int indexOfMethod(const char *method) const;
    int indexOfSignal(const char *signal) const;
    int indexOfSlot(const char *slot) const;
    int indexOfEnumerator(const char *name) const;
    int indexOfProperty(const char *name) const;
    int indexOfClassInfo(const char *name) const;
    QMetaMethod method(int index) const;
    QMetaEnum enumerator(int index) const;
    QMetaProperty property(int index) const;
    QMetaClassInfo classInfo(int index) const;
    QMetaProperty userProperty() const;
    static bool checkConnectArgs(const char *signal, const char *method);
    static QByteArray normalizedSignature(const char *method);
    static QByteArray normalizedType(const char *type);

    // internal index-based connect
    static bool connect(const QObject *sender, int signal_index,
                       const QObject *receiver, int method_index,
                       int type = 0, int *types = 0);
    // internal index-based disconnect
    static bool disconnect(const QObject *sender, int signal_index,
                         const QObject *receiver, int method_index);
    // internal slot-name based connect
    static void connectSlotsByName(QObject *o);
    // internal index-based signal activation
    static void activate(QObject *sender, int signal_index, void **argv);
    static void activate(QObject *sender, int from_signal_index, int to_signal_index, void **argv);
    static void activate(QObject *sender, const QMetaObject *, int local_signal_index, void **argv);
    static void activate(QObject *sender, const QMetaObject *, int from_local_signal_index,
      int to_local_signal_index, void
**argv);
    // internal guarded pointers
    static void addGuard(QObject **ptr);
    static void removeGuard(QObject **ptr);
    static void changeGuard(QObject **ptr, QObject *o);
    static bool invokeMethod(QObject *obj, const char *member,
                            Qt::ConnectionType,
                            QGenericReturnArgument ret,
                            QGenericArgument val0 = QGenericArgument(0),
                            QGenericArgument val1 = QGenericArgument(),
                            QGenericArgument val2 = QGenericArgument(),
                            QGenericArgument val3 = QGenericArgument(),
                            QGenericArgument val4 = QGenericArgument(),
                            QGenericArgument val5 = QGenericArgument(),
                            QGenericArgument val6 = QGenericArgument(),
                            QGenericArgument val7 = QGenericArgument(),
                            QGenericArgument val8 = QGenericArgument(),
                            QGenericArgument val9 = QGenericArgument());
    // [ ... several invokeMethod overloads ...]
    enum Call {
        InvokeMetaMethod,
        ReadProperty,
        WriteProperty,
        ResetProperty,
        QueryPropertyDesignable,
        QueryPropertyScriptable,
        QueryPropertyStored,
        QueryPropertyEditable,
        QueryPropertyUser
    };
}
```
