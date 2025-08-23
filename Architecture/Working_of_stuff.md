# Optical Drives
## Writing Process on a Disc
A recordable disc has several layers, but the key one is the **recording layer**. This layer is made of a special **organic dye** (for CD-R, DVD-R, BD-R) or a **phase-change alloy** (for RW/RE discs).

The burner in your drive has a powerful laser, much stronger than the one used just for reading.

*   **For Write-Once Discs (R):** The laser is tuned to a specific power that heats up the organic dye layer in a tiny, precise spot. This heat **chemically alters** the dye, making it opaque (dark).
*   For **Rewritable Discs (RW/RE):** The laser heats a special crystalline alloy layer. Depending on the temperature and cooling speed, the drive can change the material's state:
    *   **To Write (Crystalline -> Amorphous):** High heat melts the material, and it's cooled so quickly it solidifies in a non-crystalline (amorphous) state. This spot becomes less reflective.
    *   **To Erase (Amorphous -> Crystalline):** The laser applies a lower level of heat that is just enough to anneal the material, allowing it to cool back into its natural, reflective crystalline state, effectively "erasing" the spot.


## How Multi-Layered Discs Work

In a multi-layered disc, there are two or more separate data layers, each on its own plane. The key is that the first layer (L0) you encounter is **semi-reflective**. It reflects just enough light to be read, but also allows a significant portion of the laser light to **pass through** it to the layer beneath (L1), which is fully reflective.

*   The reader/burner has an actuator that can minutely change the **focus** of the laser beam.
*   To read the **top layer** (L0), the lens focuses the laser beam on that plane. The light reflects off the semi-transparent layer and is read by the sensor.
*   To read the **deeper layer** (L1), the lens physically adjusts its position to change the focal point of the beam, focusing it deeper onto the second layer. The light now passes through the semi-transparent first layer, reflects off the fully reflective second layer, passes back through the first layer, and is read by the sensor.

![](https://www.techspot.com/articles-info/1986/images/2020-02-20-image-2-p_1100.webp)