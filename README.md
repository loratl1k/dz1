convex.py:
```
from deq import Deq
from r2point import R2Point


class Figure:
    """ Абстрактная фигура """

    def perimeter(self):
        return 0.0

    def area(self):
        return 0.0


class Void(Figure):
    """ "Hульугольник" """

    def add(self, p):
        return Point(p)


class Point(Figure):
    """ "Одноугольник" """

    def __init__(self, p):
        self.p = p

    def add(self, q):
        return self if self.p == q else Segment(self.p, q)


class Segment(Figure):
    """ "Двуугольник" """

    def __init__(self, p, q):
        self.p, self.q = p, q

    def perimeter(self):
        return 2.0 * self.p.dist(self.q)

    def add(self, r):
        if R2Point.is_triangle(self.p, self.q, r):
            return Polygon(self.p, self.q, r)
        elif self.q.is_inside(self.p, r):
            return Segment(self.p, r)
        elif self.p.is_inside(r, self.q):
            return Segment(r, self.q)
        else:
            return self


class Polygon(Figure):
    """ Многоугольник """

    def __init__(self, a, b, c):
        self.points = Deq()
        self.points.push_first(b)
        if b.is_light(a, c):
            self.points.push_first(a)
            self.points.push_last(c)
        else:
            self.points.push_last(a)
            self.points.push_first(c)
        self._perimeter = a.dist(b) + b.dist(c) + c.dist(a)
        self._area = abs(R2Point.area(a, b, c))

    def perimeter(self):
        return self._perimeter

    def area(self):
        return self._area

    # добавление новой точки
    def add(self, t):

        # поиск освещённого ребра
        for n in range(self.points.size()):
            if t.is_light(self.points.last(), self.points.first()):
                break
            self.points.push_last(self.points.pop_first())

        # хотя бы одно освещённое ребро есть
        if t.is_light(self.points.last(), self.points.first()):

            # учёт удаления ребра, соединяющего конец и начало дека
            self._perimeter -= self.points.first().dist(self.points.last())
            self._area += abs(R2Point.area(t,
                                           self.points.last(),
                                           self.points.first()))

            # удаление освещённых рёбер из начала дека
            p = self.points.pop_first()
            while t.is_light(p, self.points.first()):
                self._perimeter -= p.dist(self.points.first())
                self._area += abs(R2Point.area(t, p, self.points.first()))
                p = self.points.pop_first()
            self.points.push_first(p)

            # удаление освещённых рёбер из конца дека
            p = self.points.pop_last()
            while t.is_light(self.points.last(), p):
                self._perimeter -= p.dist(self.points.last())
                self._area += abs(R2Point.area(t, p, self.points.last()))
                p = self.points.pop_last()
            self.points.push_last(p)

            # добавление двух новых рёбер
            self._perimeter += t.dist(self.points.first()) + \
                t.dist(self.points.last())
            self.points.push_first(t)

        return self


if __name__ == "__main__":
    f = Void()
    print(type(f), f.__dict__)
    f = f.add(R2Point(0.0, 0.0))
    print(type(f), f.__dict__)
    f = f.add(R2Point(1.0, 0.0))
    print(type(f), f.__dict__)
    f = f.add(R2Point(0.0, 1.0))
    print(type(f), f.__dict__)
```

deq.py:
```
class Deq:
    """
    Реализация дека на базе вектора
    для языка Python она тривиальна)
    """
    # Конструктор

    def __init__(self):
        self.array = []

    # Количество элементов в деке
    def size(self):
        return len(self.array)

    # Добавить элемент в конец дека
    def push_last(self, c):
        self.array.append(c)

    # Добавить элемент в начало дека
    def push_first(self, c):
        self.array.insert(0, c)

    # Удалить элемент из конца дека
    def pop_last(self):
        return self.array.pop()

    # Удалить элемент из начала дека
    def pop_first(self):
        return self.array.pop(0)

    # Последний элемент дека
    def last(self):
        return self.array[len(self.array) - 1]

    # Первый элемент дека
    def first(self):
        return self.array[0]

    def len(self):
        return len(self.array)


if __name__ == "__main__":
    s = Deq()
    print(s.__dict__)
    s.push_first(1)
    s.push_first(2)
    print(s.__dict__)
    a = s.pop_last()
    print(f"a={a}, array={s.__dict__}")
```

my.py:
```
from deq import Deq
from r2point import R2Point
from convex import Void


class Quarter:

    def area(self):
        return 0.0


class Zero(Quarter):

    def __init__(self):
        self.r = []

    def add(self, t):
        return One(self.r, t)


class One(Quarter):

    def __init__(self, r, t):
        self.t = t
        self.r = r
        if t.quad() is True:
            self.r.append(t)

    def add(self, p):
        return self if p == self.t else Two(self.r, self.t, p)


class Two(Quarter):

    def __init__(self, r, t, p):
        self.t = t
        self.r = r
        self.p = p
        if p.quad() is True:
            self.r.append(p)
        a = R2Point.ox(p, t)
        if a.is_inside(self.t, self.p):
            if a.quad() is True:
                self.r.append(a)
        b = R2Point.oy(p, t)
        if b.is_inside(self.t, self.p):
            if b.quad() is True:
                self.r.append(b)

    def add(self, q):
        if R2Point.is_triangle(self.p, self.t, q):
            return Three(self.r, self.t, self.p, q)
        elif self.t.is_inside(self.p, q):
            return Two(self.r, self.p, q)
        elif self.p.is_inside(q, self.t):
            return Two(self.r, q, self.t)
        else:
            return self


class Three(Quarter):

    def __init__(self, r, t, p, q):
        self.q = q
        self.t = t
        self.r = r
        self.p = p
        self.h = Void()
        if q.quad() is True:
            self.r.append(q)
        a = R2Point.ox(self.q, self.t)
        if a.is_inside(self.q, self.t):
            if a.quad() is True:
                self.r.append(a)
        b = R2Point.oy(self.q, self.t)
        if b.is_inside(self.q, self.t):
            if b.quad() is True:
                self.r.append(b)
        d = R2Point.ox(self.q, self.p)
        if d.is_inside(self.q, self.p):
            if d.quad() is True:
                self.r.append(d)
        v = R2Point.oy(self.q, self.p)
        if v.is_inside(self.q, self.p):
            if v.quad() is True:
                self.r.append(v)
        if R2Point.tri(self.t, self.p, self.q):
            c = R2Point(0, 0)
            self.r.append(c)
        self.points = Deq()
        self.points.push_first(p)
        if b.is_light(t, q):
            self.points.push_first(t)
            self.points.push_last(q)
        else:
            self.points.push_last(t)
            self.points.push_first(q)
        for i in self.r:
            self.h = self.h.add(i)
        self._area = self.h.area()

    def add(self, t):
        for n in range(self.points.size()):
            if t.is_light(self.points.last(), self.points.first()):
                break
            self.points.push_last(self.points.pop_first())

        if t.is_light(self.points.last(), self.points.first()):

            # удаление освещённых рёбер из начала дека
            p = self.points.pop_first()
            while t.is_light(p, self.points.first()):
                p = self.points.pop_first()
            self.points.push_first(p)

            # удаление освещённых рёбер из конца дека
            p = self.points.pop_last()
            while t.is_light(self.points.last(), p):
                p = self.points.pop_last()
            self.points.push_last(p)

            self.points.push_first(t)

        if t.quad() is True:
            self.h = self.h.add(t)
            self._area = self.h.area()

        for n in range(self.points.size()):
            a = R2Point.ox(t, self.points.first())
            if a.is_inside(t, self.points.first()):
                if a.quad() is True:
                    self.h = self.h.add(a)
                    self._area = self.h.area()
            b = R2Point.oy(t, self.points.first())
            if b.is_inside(t, self.points.first()):
                if b.quad() is True:
                    self.h = self.h.add(b)
                    self._area = self.h.area()
            if R2Point.tri(self.t, self.points.last(), self.points.first()):
                c = R2Point(0, 0)
                self.h = self.h.add(c)
                self._area = self.h.area()
            self.points.push_last(self.points.pop_first())

        return self

    def area(self):
        return self._area


if __name__ == "__main__":
    f = Void()
    print(type(f), f.__dict__)
    f = f.add(R2Point(0.0, 0.0))
    print(type(f), f.__dict__)
    f = f.add(R2Point(1.0, 0.0))
    print(type(f), f.__dict__)
    f = f.add(R2Point(0.0, 1.0))
    print(type(f), f.__dict__)

```

r2point.py:
```
from math import sqrt
from math import copysign


class R2Point:
    """ Точка (Point) на плоскости (R2) """

    # Конструктор
    def __init__(self, x=None, y=None):
        if x is None:
            x = float(input("x -> "))
        if y is None:
            y = float(input("y -> "))
        self.x, self.y = x, y

    # Площадь треугольника
    @staticmethod
    def area(a, b, c):

        return 0.5 * ((a.x - c.x) * (b.y - c.y) - (a.y - c.y) * (b.x - c.x))

    # Лежат ли точки на одной прямой?
    @staticmethod
    def is_triangle(a, b, c):
        return R2Point.area(a, b, c) != 0.0

    # Расстояние до другой точки
    def dist(self, other):
        return sqrt((other.x - self.x) ** 2 + (other.y - self.y) ** 2)

    # Лежит ли точка внутри "стандартного" прямоугольника?
    def is_inside(self, a, b):
        return (((a.x <= self.x and self.x <= b.x) or
                 (a.x >= self.x and self.x >= b.x)) and
                ((a.y <= self.y and self.y <= b.y) or
                 (a.y >= self.y and self.y >= b.y)))

    def quad(self):
        if self.x >= 0.0 and self.y >= 0.0:
            return True
        else:
            return False

    @staticmethod
    def tri(a, b, c):
        if (copysign(1, a.x * (b.y - a.y) - (b.x - a.x) * a.y) ==
                copysign(1, b.x * (c.y - b.y) - (c.x - b.x) * b.y) and
                copysign(1, a.x * (b.y - a.y) - (b.x - a.x) * a.y) ==
                copysign(1, c.x * (a.y - c.y) - (a.x - c.x) * c.y) and
                copysign(1, b.x * (c.y - b.y) - (c.x - b.x) * b.y) ==
                copysign(1, c.x * (a.y - c.y) - (a.x - c.x) * c.y)):
            return True

        else:
            return False

    @staticmethod
    def oy(t, p):
        if p.y == t.y:
            a = R2Point(p.y, 0)
            return a
        else:
            b = R2Point(t.x - (t.y * (p.x - t.x)) / (p.y - t.y), 0)
            return b

    @staticmethod
    def ox(t, p):
        if p.x == t.x:
            a = R2Point(0, p.x)
            return a
        else:
            b = R2Point(0, t.y - (t.x * (p.y - t.y)) / (p.x - t.x))
            return b

    # Освещено ли из данной точки ребро (a,b)?
    def is_light(self, a, b):
        s = R2Point.area(a, b, self)
        return s < 0.0 or (s == 0.0 and not self.is_inside(a, b))

    # Совпадает ли точка с другой?
    def __eq__(self, other):
        if isinstance(other, type(self)):
            return self.x == other.x and self.y == other.y
        return False


if __name__ == "__main__":
    x = R2Point(1.0, 1.0)
    print(type(x), x.__dict__)
    print(x.dist(R2Point(1.0, 0.0)))
    a, b, c = R2Point(0.0, 0.0), R2Point(1.0, 0.0), R2Point(1.0, 1.0)
    print(R2Point.area(a, c, b))
```

run_convex.py:
```
#!/usr/bin/env -S python3 -B
from r2point import R2Point
from convex import Void
from my import Zero

h = Zero()
f = Void()
try:
    while True:
        x = int(input("x->"))
        y = int(input("y->"))
        f = f.add(R2Point(x, y))
        h = h.add(R2Point(x, y))
        print(f"S = {f.area()}, P = {f.perimeter()}, Smod = {h.area()}")
        print()
except(EOFError, KeyboardInterrupt):
    print("\nStop")
```

run_tk_convex.py
```
#!/usr/bin/env -S python3 -B
from tk_drawer import TkDrawer
from r2point import R2Point
from convex import Void, Point, Segment, Polygon
from my import Zero


def void_draw(self, tk):
    pass


def point_draw(self, tk):
    tk.draw_point(self.p)


def segment_draw(self, tk):
    tk.draw_line(self.p, self.q)


def polygon_draw(self, tk):
    for n in range(self.points.size()):
        tk.draw_line(self.points.last(), self.points.first())
        self.points.push_last(self.points.pop_first())


setattr(Void, 'draw', void_draw)
setattr(Point, 'draw', point_draw)
setattr(Segment, 'draw', segment_draw)
setattr(Polygon, 'draw', polygon_draw)


tk = TkDrawer()
f = Void()
h = Zero()

tk.clean()

try:
    while True:
        x = int(input("x->"))
        y = int(input("y->"))
        f = f.add(R2Point(x, y))
        h = h.add(R2Point(x, y))
        tk.clean()
        f.draw(tk)
        print(f"S = {f.area()}, P = {f.perimeter()}, Smod = {h.area()}")
except(EOFError, KeyboardInterrupt):
    print("\nStop")
    tk.close()
```

from tkinter import *

tk_drawer.py:
```
# Размер окна
SIZE = 600
# Коэффициент гомотетии
SCALE = 50


def x(p):
    """ преобразование x-координаты """
    return SIZE / 2 + SCALE * p.x


def y(p):
    """ преобразование y-координаты """
    return SIZE / 2 - SCALE * p.y


class TkDrawer:
    """ Графический интерфейс для выпуклой оболочки """

    # Конструктор
    def __init__(self):
        self.root = Tk()
        self.root.title("Выпуклая оболочка")
        self.root.geometry(f"{SIZE+5}x{SIZE+5}")
        self.root.resizable(False, False)
        self.root.bind('<Control-c>', quit)
        self.canvas = Canvas(self.root, width=SIZE, height=SIZE)
        self.canvas.pack(padx=5, pady=5)

    # Завершение работы
    def close(self):
        self.root.quit()

    # Стирание существующей картинки и рисование осей координат
    def clean(self):
        self.canvas.create_rectangle(0, 0, SIZE, SIZE, fill="white")
        self.canvas.create_line(0, SIZE / 2, SIZE, SIZE / 2, fill="blue")
        self.canvas.create_line(SIZE / 2, 0, SIZE / 2, SIZE, fill="blue")
        self.root.update()

    # Рисование точки
    def draw_point(self, p):
        self.canvas.create_oval(
            x(p) + 1, y(p) + 1, x(p) - 1, y(p) - 1, fill="black")
        self.root.update()

    # Рисование линии
    def draw_line(self, p, q):
        self.canvas.create_line(x(p), y(p), x(q), y(q), fill="black", width=2)
        self.root.update()

    # Рисование точки
    def draw_point_red(self, p):
        self.canvas.create_oval(
            x(p) + 1, y(p) + 1, x(p) - 1, y(p) - 1, fill="red")
        self.root.update()

    # Рисование линии
    def draw_line_red(self, p, q):
        self.canvas.create_line(x(p), y(p), x(q), y(q), fill="red", width=2)
        self.root.update()


if __name__ == "__main__":

    import time
    from r2point import R2Point
    tk = TkDrawer()
    tk.clean()
    tk.draw_point(R2Point(2.0, 2.0))
    tk.draw_line(R2Point(0.0, 0.0), R2Point(1.0, 1.0))
    tk.draw_line(R2Point(0.0, 0.0), R2Point(1.0, 0.0))
    time.sleep(5)
```

test_convex_and_my.py:
```
from pytest import approx
from math import sqrt
from r2point import R2Point
from convex import Figure, Void, Point, Segment, Polygon
from my import Quarter, Zero, One, Two, Three


class TestVoid:

    # Инициализация (выполняется для каждого из тестов класса)
    def setup_method(self):
        self.f = Void()

    # Нульугольник является фигурой
    def test_figure(self):
        assert isinstance(self.f, Figure)

    # Конструктор порождает экземпляр класса Void (нульугольник)
    def test_void(self):
        assert isinstance(self.f, Void)

    # Периметр нульугольника нулевой
    def test_perimeter(self):
        assert self.f.perimeter() == 0.0

    # Площадь нульугольника нулевая
    def test_аrea(self):
        assert self.f.area() == 0.0

    # При добавлении точки нульугольник превращается в одноугольник
    def test_add(self):
        assert isinstance(self.f.add(R2Point(0.0, 0.0)), Point)


class TestPoint:

    # Инициализация (выполняется для каждого из тестов класса)
    def setup_method(self):
        self.f = Point(R2Point(0.0, 0.0))

    # Одноугольник является фигурой
    def test_figure(self):
        assert isinstance(self.f, Figure)

    # Конструктор порождает экземпляр класса Point (одноугольник)
    def test_point(self):
        assert isinstance(self.f, Point)

    # Периметр одноугольника нулевой
    def test_perimeter(self):
        assert self.f.perimeter() == 0.0

    # Площадь одноугольника нулевая
    def test_аrea(self):
        assert self.f.area() == 0.0

    # При добавлении точки одноугольник может не измениться
    def test_add1(self):
        assert self.f.add(R2Point(0.0, 0.0)) is self.f

    # При добавлении точки одноугольник может превратиться в двуугольник
    def test_add2(self):
        assert isinstance(self.f.add(R2Point(1.0, 0.0)), Segment)


class TestSegment:

    # Инициализация (выполняется для каждого из тестов класса)
    def setup_method(self):
        self.f = Segment(R2Point(0.0, 0.0), R2Point(1.0, 0.0))

    # Двуугольник является фигурой
    def test_figure(self):
        assert isinstance(self.f, Figure)

    # Конструктор порождает экземпляр класса Segment (двуугольник)
    def test_segment(self):
        assert isinstance(self.f, Segment)

    # Периметр двуугольника равен удвоенной длине отрезка
    def test_perimeter(self):
        assert self.f.perimeter() == approx(2.0)

    # Площадь двуугольника нулевая
    def test_аrea(self):
        assert self.f.area() == 0.0

    # При добавлении точки двуугольник может не измениться
    def test_add1(self):
        assert self.f.add(R2Point(0.5, 0.0)) is self.f

    # При добавлении точки двуугольник может превратиться в другой двуугольник
    def test_add2(self):
        assert isinstance(self.f.add(R2Point(2.0, 0.0)), Segment)

    # При добавлении точки двуугольник может превратиться в треугольник
    def test_add3(self):
        assert isinstance(self.f.add(R2Point(0.0, 1.0)), Polygon)


class TestPolygon:

    # Инициализация (выполняется для каждого из тестов класса)
    def setup_method(self):
        self.f = Polygon(
            R2Point(
                0.0, 0.0), R2Point(
                1.0, 0.0), R2Point(
                0.0, 1.0))

    # Многоугольник является фигурой
    def test_figure(self):
        assert isinstance(self.f, Figure)

    # Конструктор порождает экземпляр класса Polygon (многоугольник)
    def test_polygon(self):
        assert isinstance(self.f, Polygon)

    # Изменение количества вершин многоугольника
    #   изначально их три
    def test_vertexes1(self):
        assert self.f.points.size() == 3
    #   добавление точки внутрь многоугольника не меняет их количества

    def test_vertexes2(self):
        assert self.f.add(R2Point(0.1, 0.1)).points.size() == 3
    #   добавление другой точки может изменить их количество

    def test_vertexes3(self):
        assert self.f.add(R2Point(1.0, 1.0)).points.size() == 4
    #   изменения выпуклой оболочки могут и уменьшать их количество

    def test_vertexes4(self):
        assert self.f.add(
            R2Point(
                0.4,
                1.0)).add(
            R2Point(
                1.0,
                0.4)).add(
                    R2Point(
                        0.8,
                        0.9)).add(
                            R2Point(
                                0.9,
                                0.8)).points.size() == 7
        assert self.f.add(R2Point(2.0, 2.0)).points.size() == 4

    # Изменение периметра многоугольника
    #   изначально он равен сумме длин сторон
    def test_perimeter1(self):
        assert self.f.perimeter() == approx(2.0 + sqrt(2.0))
    #   добавление точки может его изменить

    def test_perimeter2(self):
        assert self.f.add(R2Point(1.0, 1.0)).perimeter() == approx(4.0)

    # Изменение площади многоугольника
    #   изначально она равна (неориентированной) площади треугольника
    def test_аrea1(self):
        assert self.f.area() == approx(0.5)
    #   добавление точки может увеличить площадь

    def test_area2(self):
        assert self.f.add(R2Point(1.0, 1.0)).area() == approx(1.0)


class TestZero:

    def setup_method(self):
        self.f = Zero()

    def test_quarter(self):
        assert isinstance(self.f, Quarter)

    def test_void(self):
        assert isinstance(self.f, Zero)

    def test_аrea(self):
        assert self.f.area() == 0.0

    def test_add(self):
        assert isinstance(self.f.add(R2Point(0.0, 0.0)), One)


class TestOne:

    def setup_method(self):
        self.r = [R2Point(0.0, 0.0)]
        self.f = One(self.r, R2Point(0.0, 0.0))

    def test_quarter(self):
        assert isinstance(self.f, Quarter)

    def test_point(self):
        assert isinstance(self.f, One)

    def test_аrea(self):
        assert self.f.area() == 0.0

    def test_add1(self):
        assert self.f.add(R2Point(0.0, 0.0)) is self.f

    def test_add2(self):
        assert isinstance(self.f.add(R2Point(1.0, 0.0)), Two)


class TestTwo:

    def setup_method(self):
        self.r = [R2Point(0.0, 0.0), R2Point(1.0, 0.0)]
        self.f = Two(self.r, R2Point(0.0, 0.0), R2Point(1.0, 0.0))

    def test_figure(self):
        assert isinstance(self.f, Quarter)

    def test_two(self):
        assert isinstance(self.f, Two)

    def test_аrea(self):
        assert self.f.area() == 0.0

    def test_add1(self):
        assert self.f.add(R2Point(0.5, 0.0)) is self.f

    def test_add2(self):
        assert isinstance(self.f.add(R2Point(2.0, 0.0)), Two)

    def test_add3(self):
        assert isinstance(self.f.add(R2Point(0.0, 1.0)), Three)


class TestThree:

    def setup_method(self):
        self.r = [R2Point(
                0.0, 0.0), R2Point(
                1.0, 0.0), R2Point(
                0.0, 1.0)]
        self.f = Three(self.r,
                       R2Point(0.0, 0.0),
                       R2Point(1.0, 0.0),
                       R2Point(0.0, 1.0))

    def test_quarter(self):
        assert isinstance(self.f, Quarter)

    def test_three(self):
        assert isinstance(self.f, Three)

    def test_аrea1(self):
        assert self.f.area() == approx(0.5)

    def test_area2(self):
        assert self.f.add(R2Point(-1.0, -1.0)).area() == approx(0.5)

    def test_area3(self):
        self.r = [R2Point(0.0, 0.0)]
        self.f = Three(self.r,
                       R2Point(-1.0, 0.0),
                       R2Point(0.0, -1.0),
                       R2Point(0.0, 0.0))
        assert self.f.area() == approx(0.0)

    def test_area4(self):
        self.r = [R2Point(0.0, 0.0),
                  R2Point(0.0, 1.0),
                  R2Point(2.0, 0.0),
                  R2Point(2.0, 2.0)]
        self.f = Three(self.r,
                       R2Point(-2.0, 0.0),
                       R2Point(2.0, 2.0),
                       R2Point(2.0, -5.0))
        assert self.f.area() == approx(3.0)

    def test_area5(self):
        self.r = [R2Point(1.0, 0.0), R2Point(0.0, 1.0)]
        self.f = Three(self.r,
                       R2Point(-2.0, 3.0),
                       R2Point(-2.0, -1.0),
                       R2Point(2.0, -1.0))
        assert self.f.add(R2Point(2.0, 3.0)).area() == approx(6.0)

    def test_area6(self):
        self.r = [R2Point(1.0, 0.0), R2Point(0.0, 1.0)]
        self.f = Three(self.r,
                       R2Point(-2.0, 3.0),
                       R2Point(-2.0, -1.0),
                       R2Point(2.0, -1.0))
        self.f.add(R2Point(2.0, 3.0))
        assert self.f.add(R2Point(2.0, 5.0)).area() == approx(9.0)

    def test_area7(self):
        self.r = [R2Point(1.0, 0.0), R2Point(0.0, 1.0)]
        self.f = Three(self.r,
                       R2Point(-2.0, 3.0),
                       R2Point(-2.0, -1.0),
                       R2Point(2.0, -1.0))
        self.f.add(R2Point(2.0, 3.0))
        self.f.add(R2Point(2.0, 3.0))
        self.f.add(R2Point(1.0, 1.0))
        self.f.add(R2Point(-3.0, -2.0))
        self.f.add(R2Point(-4.0, 1.0))
        assert self.f.add(R2Point(2.0, 3.0)).area() == approx(6.0)
```

test_r2point.py:
```
from pytest import approx
from math import sqrt
from r2point import R2Point


class TestR2Point:

    # Расстояние от точки до самой себя равно нулю
    def test_dist1(self):
        a = R2Point(1.0, 1.0)
        assert a.dist(R2Point(1.0, 1.0)) == approx(0.0)

    # Расстояние между двумя различными точками положительно
    def test_dist2(self):
        a = R2Point(1.0, 1.0)
        assert a.dist(R2Point(1.0, 0.0)) == approx(1.0)

    def test_dist3(self):
        a = R2Point(1.0, 1.0)
        assert a.dist(R2Point(0.0, 0.0)) == approx(sqrt(2.0))

    # Площадь треугольника равна нулю, если все вершины совпадают
    def test_area1(self):
        a = R2Point(1.0, 1.0)
        assert R2Point.area(a, a, a) == approx(0.0)

    # Площадь треугольника равна нулю, если все вершины лежат на одной прямой
    def test_area2(self):
        a, b, c = R2Point(0.0, 0.0), R2Point(1.0, 1.0), R2Point(2.0, 2.0)
        assert R2Point.area(a, b, c) == approx(0.0)

    # Площадь треугольника положительна при обходе вершин против часовой
    # стрелки
    def test_area3(self):
        a, b, c = R2Point(0.0, 0.0), R2Point(1.0, 0.0), R2Point(1.0, 1.0)
        assert R2Point.area(a, b, c) > 0.0

    # Площадь треугольника отрицательна при обходе вершин по часовой стрелке
    def test_area4(self):
        a, b, c = R2Point(0.0, 0.0), R2Point(1.0, 0.0), R2Point(1.0, 1.0)
        assert R2Point.area(a, c, b) < 0.0

    # Точки могут лежать внутри и вне "стандартного" прямоугольника с
    # противопложными вершинами (0,0) и (2,1)
    def test_is_inside1(self):
        a, b = R2Point(0.0, 0.0), R2Point(2.0, 1.0)
        assert R2Point(1.0, 0.5).is_inside(a, b) is True

    def test_is_inside2(self):
        a, b = R2Point(0.0, 0.0), R2Point(2.0, 1.0)
        assert R2Point(1.0, 0.5).is_inside(b, a) is True

    def test_is_inside3(self):
        a, b = R2Point(0.0, 0.0), R2Point(2.0, 1.0)
        assert R2Point(1.0, 1.5).is_inside(a, b) is False

    # Ребро [(0,0), (1,0)] может быть освещено или нет из определённой точки
    def test_is_light1(self):
        a, b = R2Point(0.0, 0.0), R2Point(1.0, 0.0)
        assert R2Point(0.5, 0.0).is_light(a, b) is False

    def test_is_light2(self):
        a, b = R2Point(0.0, 0.0), R2Point(1.0, 0.0)
        assert R2Point(2.0, 0.0).is_light(a, b) is True

    def test_is_light3(self):
        a, b = R2Point(0.0, 0.0), R2Point(1.0, 0.0)
        assert R2Point(0.5, 0.5).is_light(a, b) is False

    def test_is_light4(self):
        a, b = R2Point(0.0, 0.0), R2Point(1.0, 0.0)
        assert R2Point(0.5, -0.5).is_light(a, b) is True

    def test_quad1(self):
        assert R2Point(-1.0, -2.0).quad() is False

    def test_quad2(self):
        assert R2Point(1.0, 5.0).quad() is True

    def test_oy1(self):
        a = R2Point(-1, -1)
        b = R2Point(2, 2)
        c = R2Point(0, 0)
        assert R2Point.oy(a, b) == c

    def test_oy2(self):
        a = R2Point(2, -1)
        b = R2Point(2, 2)
        c = R2Point(2, 0)
        assert R2Point.oy(a, b) == c

    def test_ox1(self):
        a = R2Point(-1, -1)
        b = R2Point(2, 2)
        c = R2Point(0, 0)
        assert R2Point.ox(a, b) == c

    def test_ox2(self):
        a = R2Point(2, 2)
        b = R2Point(-1, 2)
        c = R2Point(0, 2)
        assert R2Point.ox(a, b) == c

    def test_tri1(self):
        a = R2Point(-3, 0)
        b = R2Point(0, 3)
        c = R2Point(2, -2)
        assert R2Point.tri(a, b, c) is True

    def test_tri2(self):
        a = R2Point(-3, 0)
        b = R2Point(0, 3)
        c = R2Point(-1, -1)
        assert R2Point.tri(a, b, c) is False
```