CycloneV internals description
==============================

Routing network
---------------

The routing network follows a single-driver structure: a number of
inputs are grouped together in one place, one is selected through the
configuration, then it is amplified and used to drive a metal line.
There is also usually one bit configuration to disable the driver,
which can be all-off (probably leaving the line floating) or a
specific combination to select vcc.  The drivers correspond to a 2d
pattern in the configuration ram.  There are 70 different patterns,
configured by 1 to 18 bits and mixing 1 to 44 inputs.

The network itself can be split in two parts: the data network and the
clock network.

The data network is a grid of connections.  Horizontal lines (H14, H6
and H3, numbered by the number of tiles they span) and vertical lines
(V12, V4 and V2) helped by wire muxes (WM) connect to each over to
ensure routing over the whole surface.  Then at the tile level
tile-data dispatch (TD) nodes allow to select between the available
signals.

Generic output (GOUT) nodes then select between TD nodes to connect to
logic blocks inputs.  Logic block outputs go to Generic Input (GIN)
nodes which feed in the connections.  In addition a dedicated network,
the Loopback dispatch (LD) connects some of the outputs from the
labs/mlabs to their inputs for fast local data routing.

The clock network is more of a top-down structure.  The top structures
are Global clocks (GCLK), Regional clocks (RCLK) and Peripheral clocks
(PCLK).  They're all driven by specialized logic blocks we call Clock
Muxes (cmux).  There are two horizontal cmux in the middle of the top
and bottom borders, each driving 4 GCLK and 20 RCLK, two vertical in
the middle of the left and right borders each driving 4 GCLK and 12
RCLK, and 3 to 4 in the corners driving 6 RCLK each.  The dies
including an HPS (sx50f and sx120f) are missing the top-right cmux
plus some of the middle-of-border-driven RCLK.  That gives a total of
16 GCLK and 66 to 88 RCLK.  In addition PCLK start from HSSI blocks to
distribute serial clocks to the network.

The GCLK span the whole grid.  A RCLK spans half the grid.  A PCLK
spans a number of tiles horizontally to its right.

The second level is Sector clocks, SCLK, which spans small rectangular
zones of tiles and connect from GCLK, RCLK and PCLK.  The on the third
level, connecting from SCLK, is Horizontal clocks (HCLK) spanning
10-15 horizontal tiles and Border clocks (BCLK) rooted regularly on
the top and bottom borders.  Finally Tile clocks (TCLK) connect from
HCLK and BCLK and distribute the clocks within a tile.

In addition the PMUX nodes at the entrance of plls select between
SCLKs, and the GCLKFB and RCLKFB bring back feedback signals from the
cmux to the pll.

Inner blocks directly connect to TCLK and have internal muxes to
select between clock and data inputs for their control.  Peripheral
blocks tend to use a secondary structure composed from a TDMUX that
selects one TD between multiple ones followed by a DCMUX that selects
between the TDMUX and a TCLK so that their clock-like inputs can be
driven from either a clock or a data signal.

Most GOUT and DCMUX connected to inputs to peripheral blocks are also
provided with an optional inverter.


Inner logic blocks
------------------

LAB
^^^

The LABs are the main combinatorial and register blocks of the FPGA.
A LAB tile includes 10 sub-blocks called cells with 64 bits of LUT
splitted in 6 parts, four Flip-Flops, two 1-bit adders and a lot of
routing logic.  In addition a common control subblock selects and
dispatches clock, enable, clear, etc signals.

Carry and share chain in the order lab (x, y+1) cell 9 -> cells 0-9 ->
lab (x, u-1) cell 0.  The BTO, TTO and BYPASS muxes control the
connections in between 5-cell blocks.


.. figure:: lab-common.*
   :width: 50%

   The part of the LAB shared by all ten cells that generates the
   common signals.


.. figure:: lab-cell.*
   :width: 100%

   One of the 10 cells of the LAB.


.. figure:: lab-modes.*
   :width: 100%

   The 16 possible interconnection modes.


.. include:: gendoc/lab-dmux.rst

.. include:: gendoc/lab-dp2r.rst


MLAB
^^^^

A MLAB is a lab that can optionally be turned into a 640-bits RAM or
ROM.  The wiring is identical to the LAB, only some additional muxes
are provided to select the RAM/ROM mode.

TODO: address/data wiring in RAM/ROM mode.

.. include:: gendoc/mlab-dmux.rst



DSP
^^^

The DSP blocks provide a multiply-adder with either three 9x9, two
18x18 or one 27x27 multiply, and the 64-bits accumulator.  Its large
number of inputs and output makes it span two tiles vertically.

TODO: everything, GOUT/GIN/DCMUX mapping is done

.. include:: gendoc/dsp-dmux.rst

.. include:: gendoc/dsp-dp2r.rst



M10K
^^^^

The M10K blocks provide 10240 (256*40) bits of dual-ported rom or ram.

TODO: everything, GOUT/GIN/DCMUX mapping is done

.. include:: gendoc/m10k-dmux.rst

.. include:: gendoc/m10k-dp2r.rst



Peripheral logic blocks
-----------------------

GPIO
^^^^

The GPIO blocks connect the FPGA with the exterior through the package
pins.  Each block controls 4 pads, which are connected to up to 4
pins.

TODO: everything, GOUT/GIN/DCMUX mapping is done

.. include:: gendoc/gpio-dmux.rst

.. include:: gendoc/gpio-dp2r.rst

.. include:: gendoc/gpio-dp2p.rst


DQS16
^^^^^

The DQS16 blocks handle differential signaling protocols.  Each
supervises 4 GPIO blocks for a total of 16 signals, hence their name.

TODO: everything

.. include:: gendoc/dqs16-dmux.rst

.. include:: gendoc/dqs16-dp2p.rst


FPLL
^^^^

The Fractional PLL blocks synthesize 9 frequencies from an input with integer or fractional ratios.

TODO: everything, GOUT/GIN/DCMUX mapping is done

.. include:: gendoc/fpll-dmux.rst

.. include:: gendoc/fpll-dp2r.rst

.. include:: gendoc/fpll-dp2p.rst


CBUF
^^^^

.. include:: gendoc/cbuf-dmux.rst


CMUXCR
^^^^^^

The three or four Corner CMUX drives 3 horizontal RCLK grids and 3 vertical each.

.. include:: gendoc/cmuxcr-dmux.rst

.. include:: gendoc/cmuxcr-dp2r.rst

.. include:: gendoc/cmuxcr-dp2p.rst


CMUXHG
^^^^^^

The two Global Horizontal CMUX drive four GCLK grids each.

.. include:: gendoc/cmuxhg-dmux.rst

.. include:: gendoc/cmuxhg-dp2r.rst

.. include:: gendoc/cmuxhg-dp2p.rst


CMUXVG
^^^^^^

The two Global Vertical CMUX drive four GCLK grids each.

.. include:: gendoc/cmuxvg-dmux.rst

.. include:: gendoc/cmuxvg-dp2r.rst

.. include:: gendoc/cmuxvg-dp2p.rst


CMUXHR
^^^^^^

The two Regional Horizontal CMUX drive 12 vertical RCLK grids each, half on each side.  Six are lost when touching the HPS.

.. include:: gendoc/cmuxhr-dmux.rst

.. include:: gendoc/cmuxhr-dp2r.rst

.. include:: gendoc/cmuxhr-dp2p.rst


CMUXVR
^^^^^^

The two Global Vertical CMUX drive 20 horizontal RCLK grids each half on each side.  Ten are lost when touching the HPS.

.. include:: gendoc/cmuxvr-dmux.rst

.. include:: gendoc/cmuxvr-dp2r.rst

.. include:: gendoc/cmuxvr-dp2p.rst


CMUXP
^^^^^

The CMUXP drive two PCLK each.

.. include:: gendoc/cmuxp-dp2r.rst


CTRL
^^^^

The Control block gives access to a number of anciliary functions of the FPGA.

TODO: everything, GOUT/GIN/DCMUX mapping is done

.. include:: gendoc/ctrl-dp2r.rst


HSSI
^^^^

The High speed serial interface blocks control the
serializing/deserializing capabilities of the FPGA.

TODO: everything

.. include:: gendoc/hssi-dmux.rst


HIP
^^^

The PCIe Hard-IP blocks control the PCIe interfaces of the FPGA.

TODO: everything

.. include:: gendoc/hip-dmux.rst


DLL
^^^

The Delay-Locked loop does phase control for the DQS16.

TODO: everything

.. include:: gendoc/dll-dmux.rst

.. include:: gendoc/dll-dp2r.rst

.. include:: gendoc/dll-dp2p.rst


SERPAR
^^^^^^

Unclear yet.

TODO: everything

.. include:: gendoc/serpar-dmux.rst


LVL
^^^

The Leveling Delay Chain does something linked to the DQS16.

TODO: everything

.. include:: gendoc/lvl-dmux.rst

.. include:: gendoc/lvl-dp2p.rst


TERM
^^^^

The TERM blocks control the On-Chip Termination circuitry

TODO: everything

.. include:: gendoc/term-dmux.rst


PMA3
^^^^

The PMA3 blocks control triplets of channels used with the HSSI.

TODO: everything

.. include:: gendoc/pma3-dmux.rst


HMC
^^^

The Hardware memory controller controls sets of GPIOs to implement
modern SDR and DDR memory interfaces.  In the sx dies one of them is
taken over by the HPS.  They can be bypassed in favor of direct access
to the GPIOs.

TODO: everything, and in particular the hmc-input -> GPIO input
mapping when bypassed.

.. include:: gendoc/hmc-dmux.rst

.. include:: gendoc/hmc-dp2r.rst

.. include:: gendoc/hmc-dp2p.rst


HPS
^^^

The interface between the FPGA and the Hard processor system is done
through 37 specialized blocks of 28 different types.

TODO: everything.  GOUT/GIN/DCMUX mapping is done except for HPS_CLOCKS.

HPS_BOOT
""""""""

.. include:: gendoc/hps_boot-dp2r.rst


HPS_CLOCKS
""""""""""

.. include:: gendoc/hps_clocks-dmux.rst

.. include:: gendoc/hps_clocks-dp2p.rst


HPS_CLOCKS_RESETS
"""""""""""""""""

.. include:: gendoc/hps_clocks_resets-dp2r.rst


HPS_CROSS_TRIGGER
"""""""""""""""""

.. include:: gendoc/hps_cross_trigger-dp2r.rst


HPS_DBG_APB
"""""""""""

.. include:: gendoc/hps_dbg_apb-dp2r.rst


HPS_DMA
"""""""

.. include:: gendoc/hps_dma-dp2r.rst


HPS_FPGA2HPS
""""""""""""

.. include:: gendoc/hps_fpga2hps-dp2r.rst


HPS_FPGA2SDRAM
""""""""""""""

.. include:: gendoc/hps_fpga2sdram-dp2r.rst


HPS_HPS2FPGA
""""""""""""

.. include:: gendoc/hps_hps2fpga-dp2r.rst


HPS_HPS2FPGA_LIGHT_WEIGHT
"""""""""""""""""""""""""

.. include:: gendoc/hps_hps2fpga_light_weight-dp2r.rst


HPS_INTERRUPTS
""""""""""""""

.. include:: gendoc/hps_interrupts-dp2r.rst


HPS_JTAG
""""""""

.. include:: gendoc/hps_jtag-dp2r.rst


HPS_LOAN_IO
"""""""""""

.. include:: gendoc/hps_loan_io-dp2r.rst


HPS_MPU_EVENT_STANDBY
"""""""""""""""""""""

.. include:: gendoc/hps_mpu_event_standby-dp2r.rst


HPS_MPU_GENERAL_PURPOSE
"""""""""""""""""""""""

.. include:: gendoc/hps_mpu_general_purpose-dp2r.rst


HPS_PERIPHERAL_CAN
""""""""""""""""""
(2 blocks)

.. include:: gendoc/hps_peripheral_can-dp2r.rst


HPS_PERIPHERAL_EMAC
"""""""""""""""""""
(2 blocks)

.. include:: gendoc/hps_peripheral_emac-dp2r.rst


HPS_PERIPHERAL_I2C
""""""""""""""""""
(4 blocks)

.. include:: gendoc/hps_peripheral_i2c-dp2r.rst


HPS_PERIPHERAL_NAND
"""""""""""""""""""

.. include:: gendoc/hps_peripheral_nand-dp2r.rst


HPS_PERIPHERAL_QSPI
"""""""""""""""""""

.. include:: gendoc/hps_peripheral_qspi-dp2r.rst


HPS_PERIPHERAL_SDMMC
""""""""""""""""""""

.. include:: gendoc/hps_peripheral_sdmmc-dp2r.rst


HPS_PERIPHERAL_SPI_MASTER
"""""""""""""""""""""""""
(2 blocks)

.. include:: gendoc/hps_peripheral_spi_master-dp2r.rst


HPS_PERIPHERAL_SPI_SLAVE
""""""""""""""""""""""""
(2 blocks)

.. include:: gendoc/hps_peripheral_spi_slave-dp2r.rst


HPS_PERIPHERAL_UART
"""""""""""""""""""
(2 blocks)

.. include:: gendoc/hps_peripheral_uart-dp2r.rst


HPS_PERIPHERAL_USB
""""""""""""""""""
(2 blocks)

.. include:: gendoc/hps_peripheral_usb-dp2r.rst


HPS_STM_EVENT
"""""""""""""

.. include:: gendoc/hps_stm_event-dp2r.rst


HPS_TEST
""""""""

.. include:: gendoc/hps_test-dp2r.rst


HPS_TPIU_TRACE
""""""""""""""

.. include:: gendoc/hps_tpiu_trace-dp2r.rst


Options
-------

.. include:: gendoc/opt-dmux.rst

