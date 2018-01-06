# Implementation in Code:

```c
#define             LIS2MDL_TWI_SCL_PIN                        11
#define             LIS2MDL_TWI_SDA_PIN                        13
#define             LIS2MDL_INT_PIN                            16
#define             LIS2MDL_ADDR                               0x1E

lis2mdl_ctx_t dev_ctx;

static uint32_t lis2mdl_twi_init(void)
{
    uint32_t err_code;

    const nrf_drv_twi_config_t twi_lis_config = {
       .scl                = LIS2MDL_TWI_SCL_PIN,
       .sda                = LIS2MDL_TWI_SDA_PIN,
       .frequency          = NRF_TWI_FREQ_400K,
       .interrupt_priority = APP_IRQ_PRIORITY_HIGHEST
    };

    err_code = itracker_i2c_init(&twi_lis_config);
    if(err_code != NRF_SUCCESS)
	  {
		    return err_code;
	  }

	  return NRF_SUCCESS;
}

static int32_t platform_write(void *handle, uint8_t Reg, uint8_t *Bufp,
                              uint16_t len)
{
    uint32_t err_code;
		err_code = itracker_i2c_write(LIS2MDL_ADDR, Reg, Bufp, len);
    return err_code;
}


static int32_t platform_read(void *handle, uint8_t Reg, uint8_t *Bufp,
                             uint16_t len)
{
		uint32_t err_code;
		err_code = itracker_i2c_read(LIS2MDL_ADDR, Reg, Bufp, len);
    return err_code;
}


uint32_t lis2mdl_init(void)
{
    uint32_t err_code;


	  err_code = lis2mdl_twi_init();
    if(err_code != NRF_SUCCESS) {
      err("LIS2MDL: error while running init for sensor");
      return err_code;
    }


		dev_ctx.write_reg = platform_write;
		dev_ctx.read_reg = platform_read;

    return NRF_SUCCESS;
}

void lis2mdl_deinit()
{
		itracker_i2c_deinit();
}


void jswrap_itracker_lis2mdldata()
{
	axis3bit16_t data_raw_magnetic;
	axis1bit16_t data_raw_temperature;
	float magnetic_mG[3];
	float temperature_degC;
	uint8_t whoamI, rst;

  lis2mdl_init();


  whoamI = 0;
  lis2mdl_device_id_get(&dev_ctx, &whoamI);
  if ( whoamI != LIS2MDL_ID )
    while(1);

  lis2mdl_reset_set(&dev_ctx, PROPERTY_ENABLE);
  do {
    lis2mdl_reset_get(&dev_ctx, &rst);
  } while (rst);

  lis2mdl_block_data_update_set(&dev_ctx, PROPERTY_ENABLE);

  lis2mdl_data_rate_set(&dev_ctx, LIS2MDL_ODR_10Hz);

  lis2mdl_set_rst_mode_set(&dev_ctx, LIS2MDL_SENS_OFF_CANC_EVERY_ODR);

  lis2mdl_offset_temp_comp_set(&dev_ctx, PROPERTY_ENABLE);

  lis2mdl_operating_mode_set(&dev_ctx, LIS2MDL_CONTINUOUS_MODE);


	uint8_t cnt=10;
	uint8_t valid=0;
  while(cnt-- || valid==0)
  {

    lis2mdl_reg_t reg;
    lis2mdl_status_get(&dev_ctx, &reg.status_reg);

    if (reg.status_reg.zyxda)
    {

      memset(data_raw_magnetic.u8bit, 0x00, 3*sizeof(int16_t));
      lis2mdl_magnetic_raw_get(&dev_ctx, data_raw_magnetic.u8bit);
      magnetic_mG[0] = LIS2MDL_FROM_LSB_TO_mG( data_raw_magnetic.i16bit[0]);
      magnetic_mG[1] = LIS2MDL_FROM_LSB_TO_mG( data_raw_magnetic.i16bit[1]);
      magnetic_mG[2] = LIS2MDL_FROM_LSB_TO_mG( data_raw_magnetic.i16bit[2]);
			valid = 1;

    }
  }

  lis2mdl_deinit();

}

```
