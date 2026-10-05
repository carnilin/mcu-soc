# Adding a new slave to the OBI interface

#### In `rtl/mcu_soc_pkg.sv`:

Step 1: Increase the localparam `NumSubordinates` by 1:

```systemverilog
// N is the previous value
localparam int unsigned NumSubordinates = N + 1;
```

Step 2: Add new enumeration value to `xbar_sub_e` in `rtl/mcu_soc_pkg.sv`:

```systemverilog
typedef enum int {
  XbarSbrMem   = 0,
  XbarSbrUart  = 1,
  ...
  XbarSbrDbg   = N - 1,
  XbarSbrI2C   = N,
  XbarSbrYOUR_ADDITION = N + 1
  } xbar_sub_e;
```

Step 3: Extend value assigned to `UseSrFifoMask`:

```systemverilog
// N is the previous value, make sure the number of 1's matches N + 1
localparam bit unsigned [xbar_cfg.Subordinates-1:0] UseSrFifoMask = (N + 1)'b11111...1;
```

Step 4: Add another entry to `SrFifoDepth`:

```systemverilog
// Amount of values in array should be N + 1, where N is the previous amount of values
localparam int unsigned SrFifoDepth [xbar_cfg.Subordinates] = '{4, 4, 4, 4, 4, ..., 4};
```

Step 5: Increase mask width in call to `TYPEDEF_XBAR_CONNECTIVITY`:

```systemverilog
// N is the previous value
`TYPEDEF_XBAR_CONNECTIVITY(Connectivity, NumSubordinates, NumManagers, {{(N + 1)'b11111...1}, {(N + 1)'b11111...1}, {(N + 1)'b11111...1}});

```

#### In `rtl/mcu_soc.sv`:

Step 6: Add your device base address to the `address_map`:

```systemverilog
assign address_map[0] = '{idx: 0,   base: 32'h8000_0000, mask: 32'hffff_2000}; 
assign address_map[1] = '{idx: 1,   base: 32'h6000_0000, mask: 32'hffff_f200};
assign address_map[2] = '{idx: 2,   base: 32'h4000_0000, mask: 32'hffff_f200};
...
assign address_map[N] = '{idx: N,   base: 32'hA000_0000, mask: 32'hffff_f200};
assign address_map[N + 1] = '{idx: N + 1, base: 32'YOUR_BASE_ADDR, mask: 32'YOUR_MASK};
// N + 1 value should match the one specified in step 2
```

Step 7: Instantiate your module, connecting the clk, reset and OBI signals:

```systemverilog
// Example for a generic module that implements the OBI interface
assign obi_r_chans_sub[XbarSbrGENERIC].obi_rid = '0;
GENERIC_MODULE GENERIC_MODULE_inst (
  .clk_i  (clk),
  .rstn_i (hwsw_rstn),

  .obi_areq_i   (obi_a_chans_sub[XbarSbrGENERIC].obi_areq),
  .obi_agnt_o   (obi_agnt_signals_sub[XbarSbrGENERIC]),
  .obi_aaddr_i  (obi_a_chans_sub[XbarSbrGENERIC].obi_aadr),
  .obi_awdata_i (obi_a_chans_sub[XbarSbrGENERIC].obi_awdata),
  .obi_awe_i    (obi_a_chans_sub[XbarSbrGENERIC].obi_awe),
  .obi_abe_i    (obi_a_chans_sub[XbarSbrGENERIC].obi_abe),

  .obi_rvalid_o (obi_r_chans_sub[XbarSbrGENERIC].obi_rvalid),
  .obi_rready_i (obi_rready_signals_sub[XbarSbrGENERIC]),
  .obi_rdata_o  (obi_r_chans_sub[XbarSbrGENERIC].obi_rdata),
  .obi_rerr_o   (obi_r_chans_sub[XbarSbrGENERIC].obi_rerr)
);
```
