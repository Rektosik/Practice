Добрий день! В мене варіант 23 
Код: 
FigureParameters:

using System;
using System.Drawing;

using System;
using System.Drawing;

namespace Practice3
{
    public class FigureParameters
    {
        public int Layers { get; set; } = 2;
        public int PointCount { get; set; } = 8;
        public float Radius { get; set; } = 200;
        public Color PointColor { get; set; } = Color.MediumSlateBlue;
    }

    public class ExtendedFigureParameters : FigureParameters
    {
        public Color LayerColor { get; set; } = Color.LightGray;

        public static ExtendedFigureParameters operator *(ExtendedFigureParameters p, float scale)
        {
            return new ExtendedFigureParameters
            {
                Layers = p.Layers,
                PointCount = p.PointCount,
                Radius = p.Radius * scale,
                PointColor = p.PointColor,
                LayerColor = p.LayerColor
            };
        }

        public static ExtendedFigureParameters operator +(ExtendedFigureParameters p, int shift)
        {
            return new ExtendedFigureParameters
            {
                Layers = p.Layers + shift,
                PointCount = p.PointCount,
                Radius = p.Radius,
                PointColor = p.PointColor,
                LayerColor = p.LayerColor
            };
        }

        public static ExtendedFigureParameters operator -(ExtendedFigureParameters p, int shift)
        {
            return new ExtendedFigureParameters
            {
                Layers = Math.Max(1, p.Layers - shift),
                PointCount = p.PointCount,
                Radius = p.Radius,
                PointColor = p.PointColor,
                LayerColor = p.LayerColor
            };
        }

        public static ExtendedFigureParameters operator ++(ExtendedFigureParameters p)
        {
            p.Layers += 1;
            return p;
        }

        public static ExtendedFigureParameters operator --(ExtendedFigureParameters p)
        {
            p.Layers = Math.Max(1, p.Layers - 1);
            return p;
        }
    }
}


HexagonStarDrawer:

using System;
using System.Collections.Generic;
using System.Drawing;

namespace Practice3
{
    public class HexagonStarDrawer
    {
        public void Draw(Graphics g, FigureParameters p, Rectangle area)
        {
            PointF center = new PointF(area.Width / 2f, area.Height / 2f);
            float radiusStep = p.Radius / p.Layers;
            int count = p.PointCount;

            List<PointF[]> layers = new List<PointF[]>();

            for (int i = 1; i <= p.Layers; i++)
            {
                float r = i * radiusStep;
                var layerPoints = GetPolygonPoints(center, r, count);
                layers.Add(layerPoints);
            }

            Pen layerPen = new Pen(p is ExtendedFigureParameters ep ? ep.LayerColor : Color.LightGray);

            for (int i = 0; i < count; i++)
                for (int j = 0; j < layers.Count - 1; j++)
                    g.DrawLine(Pens.Gray, layers[j][i], layers[j + 1][i]);

            for (int i = 0; i < layers.Count; i++)
                for (int j = i + 1; j < layers.Count; j++)
                    for (int k = 0; k < count; k++)
                        g.DrawLine(layerPen, layers[i][k], layers[j][k]);

            foreach (var layer in layers)
                for (int i = 0; i < count; i++)
                    g.DrawLine(Pens.Black, layer[i], layer[(i + 1) % count]);

            foreach (var layer in layers)
                for (int i = 0; i < count; i++)
                    for (int j = i + 1; j < count; j++)
                        g.DrawLine(Pens.DarkGray, layer[i], layer[j]);

            foreach (var layer in layers)
            {
                foreach (var pt in layer)
                {
                    using (Brush b = new SolidBrush(p.PointColor))
                        g.FillEllipse(b, pt.X - 4, pt.Y - 4, 8, 8);
                }
            }

            g.FillEllipse(Brushes.LightGreen, center.X - 5, center.Y - 5, 10, 10);
            foreach (var pt in layers[0])
                g.DrawLine(Pens.Gray, center, pt);

            layerPen.Dispose();
        }

        private PointF[] GetPolygonPoints(PointF center, float radius, int count)
        {
            var points = new PointF[count];
            for (int i = 0; i < count; i++)
            {
                double angle = i * 2 * Math.PI / count - Math.PI / 2;
                points[i] = new PointF(
                    center.X + (float)(radius * Math.Cos(angle)),
                    center.Y + (float)(radius * Math.Sin(angle))
                );
            }
            return points;
        }
    }
}
SettingsForm:

using System;
using System.Drawing;
using System.Windows.Forms;

namespace Practice3
{
    public class SettingsForm : Form
    {
        private NumericUpDown layersInput, pointsInput, radiusInput;
        private Button okButton, cancelButton, colorButton, layerColorButton;
        private ColorDialog colorDialog;
        public ExtendedFigureParameters UpdatedParameters { get; private set; }

        public SettingsForm(ExtendedFigureParameters existingParams)
        {
            this.Text = "Фігура - Налаштування";
            this.Width = 300;
            this.Height = 250;

            colorDialog = new ColorDialog();
            UpdatedParameters = new ExtendedFigureParameters
            {
                Layers = existingParams.Layers,
                PointCount = existingParams.PointCount,
                Radius = existingParams.Radius,
                PointColor = existingParams.PointColor,
                LayerColor = existingParams.LayerColor
            };

            Label layersLabel = new Label { Text = "Шари:", Left = 10, Top = 20 };
            layersInput = new NumericUpDown { Left = 130, Top = 20, Width = 100, Minimum = 1, Maximum = 10, Value = existingParams.Layers };

            Label pointsLabel = new Label { Text = "Кількість точок:", Left = 10, Top = 50 };
            pointsInput = new NumericUpDown { Left = 130, Top = 50, Width = 100, Minimum = 3, Maximum = 20, Value = existingParams.PointCount };

            Label radiusLabel = new Label { Text = "Радіус:", Left = 10, Top = 80 };
            radiusInput = new NumericUpDown
            {
                Left = 130,
                Top = 80,
                Width = 100,
                Minimum = 10,
                Maximum = 10000, 
                Value = (decimal)Math.Min(existingParams.Radius, 10000) 
            };

            colorButton = new Button { Text = "Колір точок", Left = 10, Top = 110, Width = 100, BackColor = existingParams.PointColor };
            colorButton.Click += (s, e) =>
            {
                colorDialog.Color = UpdatedParameters.PointColor;
                if (colorDialog.ShowDialog() == DialogResult.OK)
                {
                    UpdatedParameters.PointColor = colorDialog.Color;
                    colorButton.BackColor = colorDialog.Color;
                }
            };

            layerColorButton = new Button { Text = "Колір шарів", Left = 130, Top = 110, Width = 100, BackColor = existingParams.LayerColor };
            layerColorButton.Click += (s, e) =>
            {
                colorDialog.Color = UpdatedParameters.LayerColor;
                if (colorDialog.ShowDialog() == DialogResult.OK)
                {
                    UpdatedParameters.LayerColor = colorDialog.Color;
                    layerColorButton.BackColor = colorDialog.Color;
                }
            };

            okButton = new Button { Text = "OK", DialogResult = DialogResult.OK, Left = 30, Width = 100, Top = 160 };
            cancelButton = new Button { Text = "Cancel", DialogResult = DialogResult.Cancel, Left = 150, Width = 100, Top = 160 };

            Controls.AddRange(new Control[]
            {
                layersLabel, layersInput,
                pointsLabel, pointsInput,
                radiusLabel, radiusInput,
                colorButton, layerColorButton,
                okButton, cancelButton
            });

            AcceptButton = okButton;
            CancelButton = cancelButton;

            this.FormClosing += (s, e) =>
            {
                if (this.DialogResult == DialogResult.OK)
                {
                    UpdatedParameters.Layers = (int)layersInput.Value;
                    UpdatedParameters.PointCount = (int)pointsInput.Value;
                    UpdatedParameters.Radius = (float)radiusInput.Value;
                }
            };
        }
    }
}

Form1:

using System;
using System.Drawing;
using System.Windows.Forms;

namespace Practice3
{
    public partial class Form1 : Form
    {
        private ExtendedFigureParameters parameters = new ExtendedFigureParameters();
        private HexagonStarDrawer drawer = new HexagonStarDrawer();

        public Form1()
        {
            InitializeComponent();
            this.Text = "Hexagon Star Viewer";
            this.Width = 600;
            this.Height = 600;

            drawPanel.Paint += drawPanel_Paint;

        }

        private void drawPanel_Paint(object sender, PaintEventArgs e)
        {
            drawer.Draw(e.Graphics, parameters, drawPanel.ClientRectangle);
        }

        private void settingsToolStripMenuItem_Click(object sender, EventArgs e)
        {
            using (var settingsForm = new SettingsForm(parameters))
            {
                if (settingsForm.ShowDialog() == DialogResult.OK)
                {
                    parameters = settingsForm.UpdatedParameters;
                    drawPanel.Invalidate();
                    drawPanel.Update();
                }
            }
        }

        private void scaleUpToolStripMenuItem_Click(object sender, EventArgs e)
        {
            parameters = parameters * 1.5f;
            drawPanel.Invalidate();
            drawPanel.Update();
        }

        private void addLayerToolStripMenuItem_Click(object sender, EventArgs e)
        {
            parameters++;
            drawPanel.Invalidate();
            drawPanel.Update();
        }

        private void removeLayerToolStripMenuItem_Click(object sender, EventArgs e)
        {
            parameters--;
            drawPanel.Invalidate();
            drawPanel.Update();
        }
        private void drawToolStripMenuItem_Click(object sender, EventArgs e)
        {

        }
    }
}

Результат коду: 
![image](https://github.com/user-attachments/assets/4371853b-b5ee-41d7-80ab-7cae4dd18d41)

![image](https://github.com/user-attachments/assets/b92c04ce-389e-4451-bece-01a6ccb0b407)

![image](https://github.com/user-attachments/assets/7dbfe36f-c8f9-4995-a8e7-7dbd1d80c130)
