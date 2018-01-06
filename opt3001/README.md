# Implementation in Code:

```c

#define             OPT3001_TWI_SDA_PIN                        21
#define             OPT3001_INT_PIN                        		 22
#define             OPT3001_TWI_SCL_PIN                        23
#define             OPT3001_ADDR                             0x44


static uint32_t opt3001_twi_init(void)
{
    uint32_t err_code;

    const nrf_drv_twi_config_t twi_config = {
       .scl                = OPT3001_TWI_SCL_PIN,
       .sda                = OPT3001_TWI_SDA_PIN,
       .frequency          = NRF_TWI_FREQ_400K,
       .interrupt_priority = APP_IRQ_PRIORITY_HIGHEST
    };

    err_code = itracker_i2c_init(&twi_config);
    if(err_code != NRF_SUCCESS)
	  {
        return err_code;
	  }

	  return NRF_SUCCESS;
}

uint32_t opt3001_init(void)
{
    uint32_t err_code;
	  uint8_t id =0;

		//init i2c
	  err_code = opt3001_twi_init();
    if(err_code != NRF_SUCCESS) {
      err("OPT3001: issue while running init for the sensor");
      return err_code;
    }
		sensorOpt3001Enable(1);
		//sensorOpt3001Test();
    return NRF_SUCCESS;
}

void opt3001_deinit()
{
		itracker_i2c_deinit();
}

void itracker_opt3001data()
{
		int count = 2;
		uint16_t light_raw_data=0;

    opt3001_init();

		while(count--)
		{
			sensorOpt3001Read(&light_raw_data);
			float light_data = sensorOpt3001Convert(light_raw_data);
      nrf_delay_ms(1000);
		}
    opt3001_deinit();

}

```
