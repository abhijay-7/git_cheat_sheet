# Matplotlib In-Depth Cheatsheet

## Installation and Setup

```bash
# Install matplotlib
pip install matplotlib

# Install with additional packages
pip install matplotlib numpy pandas seaborn

# Common imports
import matplotlib.pyplot as plt
import matplotlib as mpl
import numpy as np
```

## Basic Plot Structure and Anatomy

### Figure and Axes Hierarchy
```python
# Basic plot structure
fig, ax = plt.subplots()        # Create figure and axes
ax.plot([1, 2, 3], [4, 5, 6])  # Plot on axes
plt.show()                      # Display plot

# Figure components
fig = plt.figure(figsize=(10, 6))  # Create figure
ax = fig.add_subplot(111)          # Add subplot
# OR
fig, ax = plt.subplots(figsize=(10, 6))

# Multiple subplots
fig, axes = plt.subplots(2, 2, figsize=(10, 8))
axes[0, 0].plot([1, 2, 3], [4, 5, 6])
axes[0, 1].scatter([1, 2, 3], [4, 5, 6])
axes[1, 0].bar([1, 2, 3], [4, 5, 6])
axes[1, 1].hist([1, 2, 2, 3, 3, 3])

# Subplot parameters
fig, axes = plt.subplots(2, 2, 
                        figsize=(12, 8),
                        sharex=True,      # Share x-axis
                        sharey=True,      # Share y-axis
                        constrained_layout=True)  # Better spacing
```

### Global vs Object-Oriented Interface
```python
# Pyplot (MATLAB-style) interface
plt.plot([1, 2, 3], [4, 5, 6])
plt.xlabel('X Label')
plt.ylabel('Y Label')
plt.title('Title')
plt.show()

# Object-oriented interface (recommended)
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [4, 5, 6])
ax.set_xlabel('X Label')
ax.set_ylabel('Y Label')
ax.set_title('Title')
plt.show()
```

## Basic Plot Types

### Line Plots
```python
x = np.linspace(0, 10, 100)
y1 = np.sin(x)
y2 = np.cos(x)

# Basic line plot
fig, ax = plt.subplots()
ax.plot(x, y1)
ax.plot(x, y2)

# Line plot with customization
ax.plot(x, y1, 
        color='blue',           # Color
        linestyle='-',          # Line style: '-', '--', '-.', ':'
        linewidth=2,            # Line width
        marker='o',             # Marker: 'o', 's', '^', 'v', '<', '>', 'd', 'p', '*'
        markersize=4,           # Marker size
        markerfacecolor='red',  # Marker fill color
        markeredgecolor='black', # Marker edge color
        alpha=0.7,              # Transparency
        label='sin(x)')         # Legend label

# Multiple lines with different styles
line_styles = ['-', '--', '-.', ':']
colors = ['blue', 'red', 'green', 'orange']
for i, (style, color) in enumerate(zip(line_styles, colors)):
    ax.plot(x, np.sin(x + i*0.5), linestyle=style, color=color, 
            label=f'sin(x + {i*0.5})')

ax.legend()
ax.grid(True)
```

### Scatter Plots
```python
# Basic scatter plot
x = np.random.randn(100)
y = np.random.randn(100)
colors = np.random.rand(100)
sizes = 1000 * np.random.rand(100)

fig, ax = plt.subplots()
scatter = ax.scatter(x, y, 
                    c=colors,           # Color mapping
                    s=sizes,            # Size mapping
                    alpha=0.6,          # Transparency
                    cmap='viridis',     # Colormap
                    edgecolors='black', # Edge color
                    linewidth=0.5)      # Edge width

# Add colorbar
plt.colorbar(scatter, ax=ax, label='Color Scale')

# Scatter plot with categories
categories = ['A', 'B', 'C']
colors = ['red', 'blue', 'green']
for category, color in zip(categories, colors):
    mask = (data['category'] == category)
    ax.scatter(data.loc[mask, 'x'], data.loc[mask, 'y'], 
              c=color, label=category, alpha=0.7)

ax.legend()
```

### Bar Plots
```python
categories = ['A', 'B', 'C', 'D']
values = [23, 45, 56, 78]

# Vertical bar plot
fig, ax = plt.subplots()
bars = ax.bar(categories, values, 
             color=['red', 'blue', 'green', 'orange'],
             alpha=0.7,
             edgecolor='black',
             linewidth=1)

# Add value labels on bars
for bar, value in zip(bars, values):
    height = bar.get_height()
    ax.text(bar.get_x() + bar.get_width()/2., height + 1,
            f'{value}', ha='center', va='bottom')

# Horizontal bar plot
fig, ax = plt.subplots()
ax.barh(categories, values)

# Grouped bar plot
x = np.arange(len(categories))
width = 0.35
values2 = [20, 35, 30, 25]

fig, ax = plt.subplots()
bars1 = ax.bar(x - width/2, values, width, label='Group 1')
bars2 = ax.bar(x + width/2, values2, width, label='Group 2')

ax.set_xlabel('Categories')
ax.set_ylabel('Values')
ax.set_xticks(x)
ax.set_xticklabels(categories)
ax.legend()

# Stacked bar plot
fig, ax = plt.subplots()
ax.bar(categories, values, label='Group 1')
ax.bar(categories, values2, bottom=values, label='Group 2')
ax.legend()
```

### Histograms
```python
data = np.random.normal(0, 1, 1000)

# Basic histogram
fig, ax = plt.subplots()
n, bins, patches = ax.hist(data, 
                          bins=30,           # Number of bins
                          density=True,      # Normalize to density
                          alpha=0.7,         # Transparency
                          color='skyblue',   # Color
                          edgecolor='black', # Edge color
                          linewidth=0.5)     # Edge width

# Multiple histograms
data1 = np.random.normal(0, 1, 1000)
data2 = np.random.normal(2, 1.5, 1000)

fig, ax = plt.subplots()
ax.hist([data1, data2], 
        bins=30, 
        label=['Dataset 1', 'Dataset 2'],
        alpha=0.7,
        color=['blue', 'red'])
ax.legend()

# 2D histogram
x = np.random.randn(1000)
y = 2 * x + np.random.randn(1000)

fig, ax = plt.subplots()
h = ax.hist2d(x, y, bins=30, cmap='Blues')
plt.colorbar(h[3], ax=ax)
```

### Box Plots
```python
data = [np.random.normal(0, std, 100) for std in range(1, 4)]

# Basic box plot
fig, ax = plt.subplots()
bp = ax.boxplot(data, 
               labels=['Group 1', 'Group 2', 'Group 3'],
               patch_artist=True,    # Fill boxes
               notch=True,          # Notched boxes
               showmeans=True)      # Show means

# Customize box plot colors
colors = ['lightblue', 'lightgreen', 'lightcoral']
for patch, color in zip(bp['boxes'], colors):
    patch.set_facecolor(color)

# Violin plot (alternative to box plot)
fig, ax = plt.subplots()
parts = ax.violinplot(data, positions=[1, 2, 3], showmeans=True)
```

## Advanced Plot Types

### Contour and Surface Plots
```python
# Create grid data
x = np.linspace(-3, 3, 100)
y = np.linspace(-3, 3, 100)
X, Y = np.meshgrid(x, y)
Z = np.sin(X) * np.cos(Y)

# Contour plot
fig, ax = plt.subplots()
contour = ax.contour(X, Y, Z, levels=10, colors='black', linewidths=0.5)
ax.clabel(contour, inline=True, fontsize=8)

# Filled contour plot
contourf = ax.contourf(X, Y, Z, levels=20, cmap='viridis', alpha=0.8)
plt.colorbar(contourf, ax=ax)

# 3D surface plot
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')
surface = ax.plot_surface(X, Y, Z, cmap='viridis', alpha=0.8)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
plt.colorbar(surface, ax=ax, shrink=0.5)
```

### Heatmaps
```python
# Create sample data
data = np.random.rand(10, 12)
xlabels = [f'Col {i}' for i in range(12)]
ylabels = [f'Row {i}' for i in range(10)]

# Basic heatmap
fig, ax = plt.subplots(figsize=(10, 6))
im = ax.imshow(data, cmap='viridis', aspect='auto')

# Add labels
ax.set_xticks(np.arange(len(xlabels)))
ax.set_yticks(np.arange(len(ylabels)))
ax.set_xticklabels(xlabels)
ax.set_yticklabels(ylabels)

# Rotate x labels
plt.setp(ax.get_xticklabels(), rotation=45, ha="right", rotation_mode="anchor")

# Add text annotations
for i in range(len(ylabels)):
    for j in range(len(xlabels)):
        text = ax.text(j, i, f'{data[i, j]:.2f}',
                      ha="center", va="center", color="white")

# Add colorbar
plt.colorbar(im, ax=ax)
```

### Subplots and Layout

```python
# Complex subplot layouts
fig = plt.figure(figsize=(12, 8))

# Grid layout
gs = fig.add_gridspec(3, 3, hspace=0.3, wspace=0.3)
ax1 = fig.add_subplot(gs[0, :])      # Top row, all columns
ax2 = fig.add_subplot(gs[1, :-1])    # Middle row, first two columns
ax3 = fig.add_subplot(gs[1:, -1])    # Right column, bottom two rows
ax4 = fig.add_subplot(gs[-1, 0])     # Bottom left
ax5 = fig.add_subplot(gs[-1, 1])     # Bottom center

# Plot different data on each subplot
x = np.linspace(0, 10, 100)
ax1.plot(x, np.sin(x))
ax2.scatter(np.random.rand(50), np.random.rand(50))
ax3.bar(['A', 'B', 'C'], [1, 2, 3])
ax4.hist(np.random.randn(1000), bins=20)
ax5.pie([1, 2, 3, 4], labels=['A', 'B', 'C', 'D'])

# Subplot with shared axes
fig, axes = plt.subplots(2, 2, figsize=(10, 8), 
                        sharex=True, sharey=True)
for i, ax in enumerate(axes.flat):
    ax.plot(np.random.randn(100).cumsum())
    ax.set_title(f'Subplot {i+1}')

plt.tight_layout()
```

## Customization and Styling

### Colors and Colormaps
```python
# Color specifications
colors = ['red', 'blue', 'green']           # Named colors
colors = ['#FF0000', '#0000FF', '#00FF00']   # Hex colors
colors = [(1, 0, 0), (0, 0, 1), (0, 1, 0)]  # RGB tuples
colors = ['C0', 'C1', 'C2']                 # Default color cycle

# Colormaps
colormaps = ['viridis', 'plasma', 'inferno', 'magma',    # Sequential
             'RdYlBu', 'RdBu', 'coolwarm',              # Diverging
             'Set1', 'Set2', 'tab10']                   # Qualitative

# Using colormaps
fig, ax = plt.subplots()
scatter = ax.scatter(x, y, c=values, cmap='viridis')
plt.colorbar(scatter)

# Custom colormap
from matplotlib.colors import LinearSegmentedColormap
colors = ['red', 'yellow', 'green']
custom_cmap = LinearSegmentedColormap.from_list('custom', colors)
```

### Text and Annotations
```python
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [4, 5, 6])

# Text annotations
ax.text(2, 5, 'Peak Point', 
        fontsize=12,
        fontweight='bold',
        ha='center',        # Horizontal alignment
        va='bottom',        # Vertical alignment
        bbox=dict(boxstyle='round', facecolor='wheat', alpha=0.8))

# Arrow annotations
ax.annotate('Important Point', 
            xy=(2, 5),                    # Point to annotate
            xytext=(2.5, 5.5),           # Text location
            arrowprops=dict(arrowstyle='->', 
                           connectionstyle='arc3,rad=0.2',
                           color='red'),
            fontsize=10,
            ha='center')

# Math text (LaTeX)
ax.text(0.5, 0.5, r'$\alpha > \beta$', 
        transform=ax.transAxes,  # Use axes coordinates (0-1)
        fontsize=16)

# Title and labels with formatting
ax.set_title('Plot Title', fontsize=16, fontweight='bold', pad=20)
ax.set_xlabel('X Label', fontsize=12, fontweight='bold')
ax.set_ylabel('Y Label', fontsize=12, fontweight='bold')
```

### Legends
```python
fig, ax = plt.subplots()
line1, = ax.plot([1, 2, 3], [4, 5, 6], label='Line 1')
line2, = ax.plot([1, 2, 3], [6, 5, 4], label='Line 2')

# Basic legend
ax.legend()

# Customized legend
ax.legend(loc='upper right',           # Location
         frameon=True,                 # Frame
         framealpha=0.8,              # Frame transparency
         shadow=True,                 # Shadow
         fancybox=True,               # Fancy box
         fontsize=12,                 # Font size
         title='Legend Title',        # Title
         title_fontsize=14)           # Title font size

# Legend outside plot
ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left')

# Custom legend entries
from matplotlib.lines import Line2D
custom_lines = [Line2D([0], [0], color='red', lw=2),
                Line2D([0], [0], color='blue', lw=2)]
ax.legend(custom_lines, ['Custom 1', 'Custom 2'])
```

### Axes Customization
```python
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [4, 5, 6])

# Axis limits
ax.set_xlim(0, 4)
ax.set_ylim(0, 7)
# OR
ax.axis([0, 4, 0, 7])  # [xmin, xmax, ymin, ymax]

# Axis ticks
ax.set_xticks([1, 2, 3])
ax.set_xticklabels(['One', 'Two', 'Three'])
ax.set_yticks(np.arange(0, 8, 1))

# Tick parameters
ax.tick_params(axis='x',          # Which axis
              direction='in',     # Tick direction
              length=6,           # Tick length
              width=2,            # Tick width
              color='red',        # Tick color
              labelsize=12,       # Label size
              rotation=45)        # Label rotation

# Hide ticks/labels
ax.set_xticks([])
ax.set_xticklabels([])

# Logarithmic scale
ax.set_xscale('log')
ax.set_yscale('log')

# Axis spines
ax.spines['top'].set_visible(False)     # Hide top spine
ax.spines['right'].set_visible(False)   # Hide right spine
ax.spines['left'].set_linewidth(2)      # Thicker left spine
ax.spines['bottom'].set_color('red')    # Colored bottom spine

# Grid
ax.grid(True, 
        linestyle='--',     # Line style
        alpha=0.7,          # Transparency
        color='gray',       # Color
        linewidth=0.5)      # Width

# Minor grid
ax.grid(True, which='minor', alpha=0.3)
ax.minorticks_on()
```

## Figure Management and Output

### Figure Properties
```python
# Figure size and DPI
fig = plt.figure(figsize=(10, 6), dpi=100)

# Background color
fig.patch.set_facecolor('lightgray')

# Tight layout (automatic spacing)
plt.tight_layout()

# Adjust subplot parameters manually
plt.subplots_adjust(left=0.1,     # Left margin
                   bottom=0.1,    # Bottom margin
                   right=0.9,     # Right margin
                   top=0.9,       # Top margin
                   wspace=0.4,    # Width spacing
                   hspace=0.4)    # Height spacing
```

### Saving Figures
```python
# Basic save
plt.savefig('figure.png')

# High-quality save
plt.savefig('figure.png',
           dpi=300,               # High resolution
           bbox_inches='tight',   # Tight bounding box
           facecolor='white',     # Background color
           edgecolor='none',      # No edge color
           transparent=False,     # Not transparent
           pad_inches=0.1)        # Padding

# Different formats
plt.savefig('figure.pdf')    # PDF (vector)
plt.savefig('figure.svg')    # SVG (vector)
plt.savefig('figure.eps')    # EPS (vector)
plt.savefig('figure.jpg', quality=95)  # JPEG with quality

# Save to BytesIO (for web applications)
from io import BytesIO
buffer = BytesIO()
plt.savefig(buffer, format='png')
buffer.seek(0)
```

## Styles and Themes

### Built-in Styles
```python
# Available styles
print(plt.style.available)

# Use style
plt.style.use('seaborn-v0_8')
plt.style.use('ggplot')
plt.style.use('dark_background')

# Multiple styles
plt.style.use(['seaborn-v0_8', 'seaborn-v0_8-darkgrid'])

# Context manager
with plt.style.context('seaborn-v0_8'):
    plt.plot([1, 2, 3], [4, 5, 6])
    plt.show()
```

### Custom Styling
```python
# Custom style dictionary
custom_style = {
    'axes.linewidth': 2,
    'axes.spines.top': False,
    'axes.spines.right': False,
    'axes.grid': True,
    'grid.alpha': 0.3,
    'font.size': 12,
    'axes.labelsize': 14,
    'xtick.labelsize': 10,
    'ytick.labelsize': 10,
    'legend.fontsize': 10,
    'figure.titlesize': 16
}

# Apply custom style
mpl.rcParams.update(custom_style)

# Reset to default
mpl.rcParams.update(mpl.rcParamsDefault)
```

## Interactive Features

### Interactive Backends
```python
# Jupyter notebook magic commands
%matplotlib inline      # Static plots inline
%matplotlib widget      # Interactive widgets
%matplotlib notebook    # Deprecated interactive

# Set backend programmatically
import matplotlib
matplotlib.use('TkAgg')  # For desktop applications
```

### Widgets and Interaction
```python
# Interactive plot with sliders (requires ipywidgets)
from ipywidgets import interact, FloatSlider

def plot_sine(amplitude=1.0, frequency=1.0):
    x = np.linspace(0, 4*np.pi, 1000)
    y = amplitude * np.sin(frequency * x)
    
    plt.figure(figsize=(10, 4))
    plt.plot(x, y)
    plt.ylim(-5, 5)
    plt.xlabel('x')
    plt.ylabel('y')
    plt.title(f'y = {amplitude} * sin({frequency} * x)')
    plt.grid(True)
    plt.show()

# Create interactive widget
interact(plot_sine,
         amplitude=FloatSlider(min=0.1, max=5.0, step=0.1, value=1.0),
         frequency=FloatSlider(min=0.1, max=3.0, step=0.1, value=1.0))
```

## Animation

### Basic Animation
```python
import matplotlib.animation as animation

# Create figure and axis
fig, ax = plt.subplots()
ax.set_xlim(0, 10)
ax.set_ylim(-2, 2)
line, = ax.plot([], [], 'b-')

# Animation function
def animate(frame):
    x = np.linspace(0, 10, 1000)
    y = np.sin(x + frame * 0.1)
    line.set_data(x, y)
    return line,

# Create animation
anim = animation.FuncAnimation(fig, animate, frames=200, 
                              interval=50, blit=True, repeat=True)

# Save animation
anim.save('sine_wave.gif', writer='pillow', fps=30)
anim.save('sine_wave.mp4', writer='ffmpeg', fps=30)

plt.show()
```

## Best Practices and Performance

### Performance Optimization
```python
# Use object-oriented interface
fig, ax = plt.subplots()
ax.plot(data)  # Faster than plt.plot(data)

# Reduce data points for large datasets
x_large = np.linspace(0, 10, 1000000)
y_large = np.sin(x_large)
# Downsample for plotting
step = len(x_large) // 1000
ax.plot(x_large[::step], y_large[::step])

# Use rasterization for complex plots
ax.plot(x, y, rasterized=True)

# Turn off interactive mode for batch processing
plt.ioff()  # Turn off interactive mode
# ... create plots ...
plt.show()  # Display all at once
plt.ion()   # Turn back on
```

### Common Patterns
```python
# Function for consistent plot styling
def setup_plot(ax, title, xlabel, ylabel):
    ax.set_title(title, fontsize=16, fontweight='bold', pad=20)
    ax.set_xlabel(xlabel, fontsize=12, fontweight='bold')
    ax.set_ylabel(ylabel, fontsize=12, fontweight='bold')
    ax.grid(True, alpha=0.3)
    ax.spines['top'].set_visible(False)
    ax.spines['right'].set_visible(False)

# Context manager for figure management
from contextlib import contextmanager

@contextmanager
def figure_manager(figsize=(10, 6), save_path=None):
    fig, ax = plt.subplots(figsize=figsize)
    try:
        yield fig, ax
    finally:
        if save_path:
            plt.savefig(save_path, dpi=300, bbox_inches='tight')
        plt.show()

# Usage
with figure_manager(figsize=(12, 8), save_path='output.png') as (fig, ax):
    ax.plot([1, 2, 3], [4, 5, 6])
    setup_plot(ax, 'My Plot', 'X Axis', 'Y Axis')
```