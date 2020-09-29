# Qt Layouts DSL

<p align="center">
  <a href="http://hits.dwyl.com/inobelar/QLayoutsDSL">
    <img title="HitCount" src="http://hits.dwyl.com/inobelar/QLayoutsDSL.svg"/>
  </a>
</p>

C++ [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) for making layouts (or filling existing) by items (other layouts or widgets)

Single-header-only.

Language version: **c++11**

Imaginary pseudo-code (inspired by [Flutter](https://flutter.dev/)):

```txt
<QVBoxLayout> {
    <QHBoxLayout> {
        Button1,
        <QVBoxLayout> {
            Button2,
            <QHBoxLayout> { Button3, Label4 }
        }
    },

    <QHBoxLayout> { Button5, LineEdit6 },

    Button7
}
```

# Usage

1. In `*.pro` file add next line: `INCLUDEPATH += $${PWD}/path/to/QLayoutsDSL/`
2. In code use: ```#include <QLayoutsDSL/q_layouts_dsl.hpp>```

# Examples of usage

## Simple

```c++
auto* layout = make<QVBoxLayout>(
    new QPushButton("Button1", this),
    new QPushButton("Button2", this),
    new QLabel     ("Label3", this)
);
```

## Advanced

```c++
auto* layout = make<QVBoxLayout>(
    make<QHBoxLayout>(
        Button1,
        make<QVBoxLayout>(
            Button2,
            make<QHBoxLayout>( Button3, Label4 )
        )
    ),

    l_info( make<QHBoxLayout>( Button5, LineEdit6 ), 3),
    stretch(2),
    Button7,
    spacing(25)
);
```

## Extended

```c++
QVBoxLayout* vbox = new QVBoxLayout;
add(vbox,
    w_info(new QPushButton("Stretch: 3", this), 3),
    w_info(new QPushButton("Stretch: 2", this), 2, Qt::AlignRight),
    w_info(new QPushButton("Stretch: 1", this), 1)
);
```

## Hacky

### Before (initialization **before** filling)

```c++
inline void clearSpacing(QLayout* layout) {
    // Remove spaces between widgets and outer margin
    layout->setSpacing(0); layout->setMargin(0);
}

class MyWidget
{
    QVBoxLayout* vbox1 = nullptr;
    QHBoxLayout* hbox1 = nullptr;
    QVBoxLayout* vbox2 = nullptr;
    QHBoxLayout* hbox2 = nullptr;

    MyWidget()
    {
        vbox1 = new QVBoxLayout; clearSpacing(vbox1);
        hbox1 = new QHBoxLayout; clearSpacing(hbox1);
        vbox2 = new QVBoxLayout; clearSpacing(vbox2);
        hbox2 = new QHBoxLayout; clearSpacing(hbox2);

        using namespace dsl;

        add(vbox1,
            new QPushButton("Button 1", this),
            add(hbox1,
                new QPushButton("Button 2", this),

                add(vbox2,
                    new QPushButton("Button 3", this),

                    add(hbox2,
                        new QPushButton("Button 4", this)
                    )
                )
            )
        );

        this->setLayout(vbox1);
    }
}
```

### After (initialization **during** filling)

```c++
template <typename LayoutT>
class ZeroLayout: public LayoutT
{
public:
    template <typename ... Args>
    explicit ZeroLayout(Args&& ... args)
        : LayoutT(args ...)
    {
        this->setMargin(0);
        this->setSpacing(0);
    }
};

class MyWidget
{
    QVBoxLayout* vbox1 = nullptr;
    QHBoxLayout* hbox1 = nullptr;
    QVBoxLayout* vbox2 = nullptr;
    QHBoxLayout* hbox2 = nullptr;

    MyWidget()
    {
        using namespace dsl;

        vbox1 = make< ZeroLayout<QVBoxLayout> >
        (
            new QPushButton("Button 1", this),

            hbox1 = make< ZeroLayout<QHBoxLayout> >
            (
                new QPushButton("Button 2", this),

                vbox2 = make< ZeroLayout<QVBoxLayout> >
                (
                    new QPushButton("Button 3", this),

                    hbox2 = make< ZeroLayout<QHBoxLayout> >
                    (
                        new QPushButton("Button 4", this)
                    )
                )
            )
        );

        this->setLayout(vbox1);
    }
}
```

## Additional usage

```c++
QVBoxLayout* vbox1 = nullptr;
QHBoxLayout* hbox1 = nullptr;
QVBoxLayout* vbox2 = nullptr;
QHBoxLayout* hbox2 = nullptr;


QPushButton* btn = new QPushButton("new", this);
btn->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Expanding);

{
    using namespace dsl;

    vbox1 = make_ex< QVBoxLayout >
    (
        new QPushButton("Button 1", this),

        hbox1 = make< QHBoxLayout >
        (
            new QPushButton("Button 2", this),

            vbox2 = make< QVBoxLayout >
            (
                new QPushButton("Button 3", this),

                hbox2 = make< QHBoxLayout >
                (
                    new QPushButton("Button 4", this)
                )
            )
        ),

        add(btn).stretch(2).alignment(Qt::AlignLeft) // <- Extra feature

    ).margin(2).spacing(5).contentsMargins(2, 20, 2, 100); // <- Extra feature
}
```
