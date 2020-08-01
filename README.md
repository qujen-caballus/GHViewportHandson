# GHViewportHandson
<h2>GrasshopperViewPortHandson for Imeditate</h2>
<h3>About Handson</h3>
<div>
  <p>
    This hands-on organized by qujen, and by Hands-onTokyoAECIndustryDevGroup on August 1.<br /> 
    Here are some of the Handsons I have held in the past.<br />
    #ここにリンクを挿
    This hands-on will be recorded and uploaded to the community discord and Youtube. I also use <a href = "https://www.deepl.com/">Deepl</a>for translation.
  </p>
</div>

<div>
  <h3>Premise</h3>
  You have a good understanding of C# and Eto, and some familiarity with Visual Studio.
  
  <h4>Preparation and My Enviroment</h4>
  <ul>
    <li>Windows10 Home</li>
    <li>Visual Studio 2019</li>
    <li>Rhinoceros6.3 or greater version</li>
    <li><a href="https://marketplace.visualstudio.com/items?itemName=McNeel.GrasshopperAssemblyforv6">Grasshopper Templates fot v6</li>
  </ul>
  
  <h4>Documentation</h4>
  <a href="http://pages.picoe.ca/docs/api/html/R_Project_EtoForms.htm">Eto</a>
  <h4>Purpose</h4>
  <ol>
    <li>learn how to hack the GH ribbon.</li>
    <li>learn how to use WPF</li>
  </ol>
  
  <h4>Deliverables</h4>
  #ここに成果物のスクショ
</div>

<h3>Let's coding!</h3>
<div>
  <h4>Add dll Filepath</h4>
  C:\Program Files\Rhino 6\System\
  <ul>
    <li>Eto.dll</li>
    <li>EtoWpf.dll</li>
    <li>Rhino.UI</li>
    <li>RhinoWindows</li>
  </ul>
  
  <h4>If using Nuget</h4>
    <p>Tools > Nuget Package Maneger > Package Maneger Console </p>
    
    PM>Find-Package Eto
    PM>Install-Package Eto.Forms -ProjectName <Your Project Name>
    PM>Install-Package Eto.Platform.Wpf -ProjectName <Your Project Name>
and more
</div>

<h4>Add Menu</h4>

  ```c#

        Eto.Forms.UITimer _timer;
        /// <summary>
        /// Waiting for the thread to end
        /// </summary>
        public void AddToMenu()
        {
            if (_timer != null)
                return;
            _timer = new Eto.Forms.UITimer();
            _timer.Interval = 1;
            _timer.Elapsed += SetupMenu;
            _timer.Start();
        }
        
        void SetupMenu(object sender, EventArgs e)
        {
            var editor = Grasshopper.Instances.DocumentEditor;

            if (null == editor || editor.Handle == IntPtr.Zero)
                return;

            var controls = editor.Controls;

            if (null == controls || controls.Count == 0)
                return;

            _timer.Stop();
            foreach (var ctrl in controls)
            {
                var menu = ctrl as Grasshopper.GUI.GH_MenuStrip;
                if (menu == null)
                    continue;
                for (int i = 0; i < menu.Items.Count; i++)
                {
                    var menuitem = menu.Items[i] as ToolStripMenuItem;
                    if (menuitem.Text == "Display")
                    {
                        for (int j = 0; j < menuitem.DropDownItems.Count; j++)
                        {
                            if (menuitem.DropDownItems[j].Text.StartsWith("canvas widgets", StringComparison.OrdinalIgnoreCase))
                            {
                                var viewportMenuItem = new ToolStripMenuItem("Viewport");
                                viewportMenuItem.CheckOnClick = true;
                                var canvasWidgets = menuitem.DropDownItems[j] as ToolStripMenuItem;
                                canvasWidgets.DropDownOpening += (s, args) =>
                                canvasWidgets.DropDownItems.Insert(0, viewportMenuItem);
                            }
                        }
                    }
                }
            }
        }
   ```
        
The exceptions are not fully handled at this point, so we'll make it look like we handled the exception

```c#
        Eto.Forms.UITimer _timer;
        Panel _viewportControlPanel;

        public void AddToMenu()
        {
            if (_timer != null)
                return;
            _timer = new Eto.Forms.UITimer();
            _timer.Interval = 1;
            _timer.Elapsed += SetupMenu;
            _timer.Start();
        }

        void SetupMenu(object sender, EventArgs e)
        {
            var editor = Grasshopper.Instances.DocumentEditor;
            if (null == editor || editor.Handle == IntPtr.Zero)
                return;

            var controls = editor.Controls;
            if (null == controls || controls.Count == 0)
                return;

            _timer.Stop();
            foreach (var ctrl in controls)
            {
                var menu = ctrl as Grasshopper.GUI.GH_MenuStrip;
                if (menu == null)
                    continue;
                for (int i = 0; i < menu.Items.Count; i++)
                {
                    var menuitem = menu.Items[i] as ToolStripMenuItem;
                    if (menuitem != null && menuitem.Text == "Display")
                    {
                        for (int j = 0; j < menuitem.DropDownItems.Count; j++)
                        {
                            if (menuitem.DropDownItems[j].Text.StartsWith("canvas widgets", StringComparison.OrdinalIgnoreCase))
                            {
                                var viewportMenuItem = new ToolStripMenuItem("Rhino Viewport");
                                viewportMenuItem.CheckOnClick = true;
                                menuitem.DropDownOpened += (s, args) =>
                                {
                                    if (_viewportControlPanel != null && _viewportControlPanel.Visible)
                                        viewportMenuItem.Checked = true;
                                    else
                                        viewportMenuItem.Checked = false;
                                };
                                var canvasWidgets = menuitem.DropDownItems[j] as ToolStripMenuItem;
                                if (canvasWidgets != null)
                                {
                                    canvasWidgets.DropDownOpening += (s, args) =>
                                        canvasWidgets.DropDownItems.Insert(0, viewportMenuItem);
                                }
                                break;
                            }
                        }
                        break;
                    }
                }
            }
        }
```

<h4>Add Viewer Panel</h4>

```c#
void ViewportMenuItem_CheckedChanged(object sender, EventArgs e)
        {var menuitem = sender as ToolStripMenuItem;
            if (menuitem != null)
            {
                if (menuitem.Checked)
                {
                    if (_viewportControlPanel == null)
                    {
                        _viewportControlPanel = new Panel();
                        _viewportControlPanel.Size = new System.Drawing.Size(400, 300);
                        _viewportControlPanel.MinimumSize = new System.Drawing.Size(50, 50);
                        _viewportControlPanel.Padding = new Padding(10);
                        Grasshopper.Instances.ActiveCanvas.Controls.Add(_viewportControlPanel);
                         }
                    _viewportControlPanel.Show();

                }
                else
                {
                    if (_viewportControlPanel != null && _viewportControlPanel.Visible)
                        _viewportControlPanel.Hide();

                }
            }
        }
```        
 
<p>
and add SetupMenu() viewportMenuItem.CheckedChanged += ViewportMenuItem_CheckedChanged </br>
It's not easy to work with as it is, so we'll fix the panel in the upper right corner. </br>
</p>

```c#
 void ViewportMenuItem_CheckedChanged(object sender, EventArgs e)
        {
            var menuitem = sender as ToolStripMenuItem;
            if (menuitem != null)
            {
                if (menuitem.Checked)
                {
                    if (_viewportControlPanel == null)
                    {
                        _viewportControlPanel = new Panel();
                        _viewportControlPanel.Size = new Size(400, 300);
                        _viewportControlPanel.MinimumSize = new Size(50, 50);
                        _viewportControlPanel.Padding = new Padding(10);

                        _viewportControlPanel.BorderStyle = BorderStyle.Fixed3D;
                        _viewportControlPanel.Location = new Point(0, 0);
                        _viewportControlPanel.Anchor = AnchorStyles.Top | AnchorStyles.Left;
                        
                        Grasshopper.Instances.ActiveCanvas.Controls.Add(_viewportControlPanel);
                        Fix(AnchorStyles.Top | AnchorStyles.Right);
                    }
                    _viewportControlPanel.Show();

                }
                else
                {
                    if (_viewportControlPanel != null && _viewportControlPanel.Visible)
                        _viewportControlPanel.Hide();

                }
            }
        }

        void Fix(AnchorStyles anchor)
        {
            FixPanel(_viewportControlPanel, anchor);
        }

        public static void FixPanel(Control ctrl, AnchorStyles anchor)
        {
            if (ctrl == null)
                return;
            var canvas = Grasshopper.Instances.ActiveCanvas;
            var canvasSize = canvas.ClientSize;
            int xEnd = 0;
            if ((anchor & AnchorStyles.Right) == AnchorStyles.Right)
                xEnd = canvasSize.Width - ctrl.Width;
            int yEnd = 0;
            if ((anchor & AnchorStyles.Bottom) == AnchorStyles.Bottom)
                yEnd = canvasSize.Height - ctrl.Height;

            ctrl.Location = new System.Drawing.Point(xEnd, yEnd);
            ctrl.Anchor = anchor;
        }
    }
```

<h4>View Rhino on Panel</h4>       
                        
create a new file (as component file) and renamed ViewPanelControl.cs.
and write codes.

```c#
 class CanvasViewportControl : RhinoWindows.Forms.Controls.ViewportControl
    {
    }
```

and call ViewportMenuItem_CheckedChanged method.

<h4>Chamge the Viewer Panel Size</h4>
<p>Generate some states by enum</p>

```c#
 enum Mode
            {
                None,
                SizeWE,
                SizeNS,
                SizeNWSE,
                SizeNESW,
                Move
            }
            Mode _mode;
```
<p>Create a method for estimating your location</p>
```c#
 Mode EstimatingLocation(Point location)
            {
                //var dock = Anchor;
                switch (Anchor)
                {
                    case (AnchorStyles.Left | AnchorStyles.Top):
                        {
                            if (location.X > (Width - Padding.Right))
                                return location.Y > (Height - Padding.Bottom) ? Mode.SizeNWSE : Mode.SizeWE;
                            if (location.Y > (Height - Padding.Bottom))
                                return Mode.SizeNS;
                            if (location.X < Padding.Left || location.Y < Padding.Top)
                                return Mode.None;
                            return Mode.None;
                        }
                    case (AnchorStyles.Left | AnchorStyles.Bottom):
                        {
                            if (location.X > (Width - Padding.Right))
                                return location.Y < Padding.Top ? Mode.SizeNESW : Mode.SizeWE;
                            if (location.Y < Padding.Top)
                                return Mode.SizeNS;
                            if (location.X < Padding.Left || location.Y > (Height - Padding.Bottom))
                                return Mode.None;
                            return Mode.None;

                        }
                    case (AnchorStyles.Right | AnchorStyles.Top):
                        {
                            if (location.X < Padding.Left)
                                return location.Y > (Height - Padding.Bottom) ? Mode.SizeNESW : Mode.SizeWE;
                            if (location.Y > (Height - Padding.Bottom))
                                return Mode.SizeNS;
                            if (location.X > (Width - Padding.Right) || location.Y < Padding.Top)
                                return Mode.None;
                            return Mode.None;
                        }
                    case (AnchorStyles.Right | AnchorStyles.Bottom):
                        {
                            if (location.X < Padding.Left)
                                return location.Y < Padding.Top ? Mode.SizeNWSE : Mode.SizeWE;
                            if (location.Y < Padding.Top)
                                return Mode.SizeNS;
                            if (location.X > (Width - Padding.Right) || location.Y > (Height - Padding.Bottom))
                                return Mode.None;
                            return Mode.None;
                        }
                }
               return Mode.None;
            }
```

<p>Write a process to change the size</p>

```c#
            protected override void OnMouseDown(MouseEventArgs e)
            {
                _mode = Mode.None;
                if (e.Button == MouseButtons.Left)
                {
                    _mode = EstimatingLocation(e.Location);
                    LeftMouseDownLocation = e.Location;
                    LeftMouseDownSize = Size;
                }
                base.OnMouseDown(e);
            }
            
            
            protected override void OnMouseMove(MouseEventArgs e)
            {
                if (_mode != Mode.None)
                {
                    int x = Location.X;
                    int y = Location.Y;
                    int width = Width;
                    int height = Height;

                    int deltaX = e.X - LeftMouseDownLocation.X;
                    int deltaY = e.Y - LeftMouseDownLocation.Y;
                    if (_mode == Mode.SizeNESW || _mode == Mode.SizeNS || _mode == Mode.SizeNWSE)
                    {
                        if ((Anchor & AnchorStyles.Top) == AnchorStyles.Top)
                            height = LeftMouseDownSize.Height + deltaY;
                        if ((Anchor & AnchorStyles.Bottom) == AnchorStyles.Bottom)
                        {
                            var pt = new Point(Location.X, Location.Y + deltaY);
                            height = Height - (pt.Y - Location.Y);
                            y = Location.Y + deltaY;
                        }
                    }
                    if (_mode == Mode.SizeNESW || _mode == Mode.SizeWE || _mode == Mode.SizeNWSE)
                    {
                        if ((Anchor & AnchorStyles.Left) == AnchorStyles.Left)
                            width = LeftMouseDownSize.Width + deltaX;
                        if ((Anchor & AnchorStyles.Right) == AnchorStyles.Right)
                        {
                            var pt = new Point(Location.X + deltaX, Location.Y);
                            width = Width - (pt.X - Location.X);
                            x = Location.X + deltaX;
                        }
                    }
                    SetBounds(x, y, width, height);
                }
                base.OnMouseMove(e);
            }
```

<p>If I don't, it will follow the mouse, so I'll write a process to change it</p>

<h4>Chage the Cursor</h4>
<p>Change the display of the mouse cursor depending on the state of the enum</p>

```c#
 public override Cursor Cursor
            {
                get
                {
                    var location = PointToClient(Control.MousePosition);
                    var mode = EstimatingLocation(location);
                    switch (mode)
                    {
                        case Mode.None:
                            return Cursors.Default;
                        case Mode.Move:
                            return Cursors.SizeAll;
                        case Mode.SizeNESW:
                            return Cursors.SizeNESW;
                        case Mode.SizeNS:
                            return Cursors.SizeNS;
                        case Mode.SizeWE:
                            return Cursors.SizeWE;
                        case Mode.SizeNWSE:
                            return Cursors.SizeNWSE;
                    }
                    return base.Cursor;
                }
                set => base.Cursor = value;
            }
```

Finish!
