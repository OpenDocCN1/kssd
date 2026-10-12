
# python 模拟 telock about window

> 原文：[`www.pediy.com/kssd/pediy11/106140.html`](https://www.pediy.com/kssd/pediy11/106140.html)

更优雅点的方法，3D to 2D 投影

代码:

```python
import wx
import random
# Formula Reference: http://membres.multimania.fr/amycode.../3dbasics.html
class Star:
    def __init__(self, starfield):
        self.starfield = starfield
        self.reset()
    def color(self):
        r = 1-self.z/self.starfield.max_z
        return (255*r, 255*r, 255*r)
    def project(self):
        sf = self.starfield
        return (self.x*sf.project_z/self.z+sf.offset_x,
                self.y*sf.project_z/self.z+sf.offset_y,)
    def reset(self):
        sf = self.starfield
        rand = random.random
        self.x = (rand()-0.5)*sf.radius
        self.y = (rand()-0.5)*sf.radius
        self.z = (rand()*sf.max_z)
        self.speed = rand()*sf.max_speed
    def move(self):
        self.z -= self.speed
        if self.z <= 0:
            self.reset()
class Starfield(list):
    def __init__(self, size, count=1000):
        self.width, self.height = size
        self.offset_x = self.width/2
        self.offset_y = self.height/2
        self.project_z = 512 # increase this Z offset to see more closer
        self.max_z = 1024
        self.max_speed = 20
        self.radius = max(self.width, self.height)
        list.__init__(self, [Star(self) for i in range(count)])
class AboutFrame(wx.Frame):
    def __init__(self):
        wx.Frame.__init__(self, None, title='for those years, for those people', size=(800, 600))
        self.starfield = Starfield(self.GetClientSize())
        self.Bind(wx.EVT_PAINT, self.OnPaint)          
        self.timer = wx.Timer(self)
        self.Bind(wx.EVT_TIMER, self.OnTimer, self.timer)          
        self.timer.Start(1000/30)#30FPS
        self.Center()
    def Draw(self, dc):
        dc.SetBackground(wx.Brush('black'))
        dc.Clear()
        
        for star in self.starfield:
            dc.SetPen(wx.Pen(star.color()))             
            dc.DrawPoint(*star.project())
            star.move()
            
    def OnTimer(self, evt):
        dc = wx.BufferedDC(wx.ClientDC(self))
        self.Draw(dc)
    def OnPaint(self, evt):
        dc = wx.BufferedPaintDC(self)
        self.Draw(dc)
if __name__ == '__main__':
    app = wx.App(redirect=False)
    AboutFrame().Show()
    app.MainLoop()
```
