# Mini Paint - Простой графический редактор на C#



![image](https://github.com/user-attachments/assets/37852a0b-200f-4c13-abc8-eb0b628595f0)


![image](https://github.com/user-attachments/assets/0ff66570-b0aa-4fe6-9dcd-8bdb767fb395)



Простой графический редактор с функционалом, аналогичным Microsoft Paint, разработанный на C# с использованием Windows Forms.

## 🖌️ Функционал

- Рисование:
  - Карандаш
  - Линии
  - Прямоугольники
  - Эллипсы
- Инструменты:
  - Ластик
  - Заливка цветом
  - Выбор цвета (ColorDialog)
  - Настройка толщины кисти
- Работа с файлами:
  - Сохранение в PNG/JPEG
  - Открытие изображений
  - Очистка холста
- Дополнительно:
  - Отмена последнего действия (Undo)
  - Предпросмотр фигур при рисовании

## 🛠️ Технологии

- C# (.NET Framework/WinForms)
- GDI+ для графических операций
- Windows Forms для интерфейса





# Код
```csharp

        private Bitmap _bitmap;
        private Graphics _graphics;
        private bool _isDrawing = false;
        private Point _lastPoint;
        private Color _currentColor = Color.Black;
        private int _brushSize = 3;
        private Tool _currentTool = Tool.Pencil;
        private Stack<Bitmap> _history = new Stack<Bitmap>();

        enum Tool { Pencil, Line, Rectangle, Ellipse, Eraser }
        public Form1()
        {
            InitializeComponent();
            _bitmap = new Bitmap(pictureBox.Width, pictureBox.Height);
            _graphics = Graphics.FromImage(_bitmap);
            _graphics.Clear(Color.White);
            pictureBox.Image = _bitmap;
        }

        private void pictureBox_MouseDown(object sender, MouseEventArgs e)
        {
            _isDrawing = true;
            _lastPoint = e.Location;
            SaveState();
        }

        private void pictureBox_MouseMove(object sender, MouseEventArgs e)
        {
            if (!_isDrawing) return;
        
            using (var g = Graphics.FromImage(_bitmap))
            {
                switch (_currentTool)
                {
                    case Tool.Pencil:
                    case Tool.Eraser:
                        using (var pen = new Pen(
                            _currentTool == Tool.Eraser ? Color.White : _currentColor,
                            _brushSize))
                        {
                            g.DrawLine(pen, _lastPoint, e.Location);
                        }
                        _lastPoint = e.Location;
                        break;

                    case Tool.Line:
                    case Tool.Rectangle: 
                    case Tool.Ellipse:
                        g.Clear(Color.White);
                        g.DrawImage(_bitmap, Point.Empty);
                        DrawShape(g, _lastPoint, e.Location);
                        break;
                }
            }
        
            pictureBox.Refresh();
        }
        private void DrawShape(Graphics g, Point start, Point end)
        {
            using (var pen = new Pen(_currentColor, _brushSize))
            {
                switch (_currentTool)
                {
                    case Tool.Line:
                        g.DrawLine(pen, start, end);
                        break;
                    case Tool.Rectangle:
                        g.DrawRectangle(pen, GetRect(start, end));
                        break;
                    case Tool.Ellipse:
                        g.DrawEllipse(pen, GetRect(start, end));
                        break;
                }
            }
        }


        private Rectangle GetRect(Point p1, Point p2)
        {
            return new Rectangle(
                Math.Min(p1.X, p2.X),
                Math.Min(p1.Y, p2.Y),
                Math.Abs(p1.X - p2.X),
                Math.Abs(p1.Y - p2.Y));
        }

        private void pictureBox_MouseUp(object sender, MouseEventArgs e)
        {
            if (!_isDrawing) return;
            _isDrawing = false;

         
            if (_currentTool != Tool.Pencil && _currentTool != Tool.Eraser)
            {
                DrawShape(_graphics, _lastPoint, e.Location);
                pictureBox.Refresh();
            }
        }

        private void btnPencil_Click(object sender, EventArgs e)
            => _currentTool = Tool.Pencil;

        private void btnLine_Click(object sender, EventArgs e)
            => _currentTool = Tool.Line;

        private void btnRectangle_Click(object sender, EventArgs e)
            => _currentTool = Tool.Rectangle;

        private void btnEllipse_Click(object sender, EventArgs e)
            => _currentTool = Tool.Ellipse;

        private void btnEraser_Click(object sender, EventArgs e)
            => _currentTool = Tool.Eraser;

     
        private void btnColor_Click(object sender, EventArgs e)
        {
            if (colorDialog.ShowDialog() == DialogResult.OK)
            {
                _currentColor = colorDialog.Color;
            }
        }
        private void trackBar1_Scroll(object sender, EventArgs e)
        {
            _brushSize = trackBar1.Value;
            lblBrushSize.Text = $"Размер: {_brushSize}px";
        }

       

        private void SaveState()
        {
            _history.Push(new Bitmap(_bitmap));
        }

        private void btnUndo_Click(object sender, EventArgs e)
        {
            if (_history.Count > 0)
            {
                _bitmap = _history.Pop();
                _graphics = Graphics.FromImage(_bitmap);
                pictureBox.Image = _bitmap;
            }
        }

        private void btnSave_Click(object sender, EventArgs e)
        {
            using (var saveDialog = new SaveFileDialog())
            {
                saveDialog.Filter = "PNG Image|*.png|JPEG Image|*.jpg";
                if (saveDialog.ShowDialog() == DialogResult.OK)
                {
                    _bitmap.Save(saveDialog.FileName);
                }
            }
        }

        private void btnOpen_Click(object sender, EventArgs e)
        {
            using (var openDialog = new OpenFileDialog())
            {
                openDialog.Filter = "Image Files|*.png;*.jpg;*.bmp";
                if (openDialog.ShowDialog() == DialogResult.OK)
                {
                    _bitmap = new Bitmap(openDialog.FileName);
                    _graphics = Graphics.FromImage(_bitmap);
                    pictureBox.Image = _bitmap;
                }
            }
        }

        private void btnClear_Click(object sender, EventArgs e)
        {
            _graphics.Clear(Color.White);
            pictureBox.Refresh();
        }

        private void btnFill_Click(object sender, EventArgs e)
        {
            using (var brush = new SolidBrush(_currentColor))
            {
                _graphics.FillRectangle(brush, new Rectangle(0, 0, _bitmap.Width, _bitmap.Height));
                pictureBox.Refresh();
            }
        }
    }
```
