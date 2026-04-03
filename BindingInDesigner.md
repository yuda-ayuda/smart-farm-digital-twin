# Tag Binding in Ignition Designer
*How to connect lifeless graphics to live data.*

When you drag a shape, label, or gauge onto an Ignition dashboard, it is just a "dead" graphic. **Binding** is the process of linking that graphic to a live Tag so it updates automatically.

### How Binding Works
Every component on your screen has a **Property Editor** panel. This panel contains the "DNA" of the component: its color, size, text, and value.

To make a component come alive:
1. Find the property that controls the main data. (e.g., For a label, it is **Text**. For a gauge, it is **Value**).
2. Look to the far right of that property's row and click the **Binding Icon** (the small chainlink).
3. Select **Tag** as the binding type.
4. Browse through the MQTT folders and select your desired tag (e.g., `lightLevel`).
5. Click **OK**. The chainlink icon will turn green, indicating the component is now "Live."
