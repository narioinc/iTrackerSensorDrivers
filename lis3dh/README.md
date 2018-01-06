# Implementation in Code:

```c

#define             LIS3DH_TWI_SCL_PIN                     18
#define             LIS3DH_TWI_SDA_PIN                     19
#define             LIS3DH_INT1_PIN                        25
#define 						LIS3DH_RES_PIN											   26
#define             LIS3DH_INT2_PIN                        27
#define             LIS3DH_ADDR                            0x19

static uint32_t lis3dh_twi_init(void)
{
    uint32_t err_code;

    const nrf_drv_twi_config_t twi_lis_config = {
       .scl                = LIS3DH_TWI_SCL_PIN,
       .sda                = LIS3DH_TWI_SDA_PIN,
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

uint32_t lis3dh_init(void)
{
    uint32_t err_code;
	  uint8_t id =0;


	  err_code = lis3dh_twi_init();
    if(err_code != NRF_SUCCESS) {
      err("LIS3DH: issue while running init for the sensor");
      return err_code;
    }

    return NRF_SUCCESS;
}

void lis3dh_deinit()
{
	itracker_i2c_deinit();
}


void jswrap_itracker_lis3dhdata()
{
		uint8_t response;
		AxesRaw_t data;
		uint8_t position=0, old_position=0;


    lis3dh_init();

		//Inizialize MEMS Sensor
		//set ODR (turn ON device)
		response = LIS3DH_SetODR(LIS3DH_ODR_100Hz);
		if(response!=1){
      err("LIS3DH: error while setting ODR");
		}
		//set PowerMode
		response = LIS3DH_SetMode(LIS3DH_NORMAL);
		if(response!=1){
      err("LIS3DH: error while setting mode");
		}
		//set Fullscale
		response = LIS3DH_SetFullScale(LIS3DH_FULLSCALE_2);
		if(response!=1){
      err("LIS3DH: error while setting full scale");
		}
		//set axis Enable
		response = LIS3DH_SetAxis(LIS3DH_X_ENABLE | LIS3DH_Y_ENABLE | LIS3DH_Z_ENABLE);
		if(response!=1){
      err("LIS3DH: error while setting Axis enable");
		}

		uint8_t cnt=10;
		while(cnt--)
		{
				//get Acceleration Raw data
				response = LIS3DH_GetAccAxesRaw(&data);
				if(response==1)
				{
						//DPRINTF(LOG_INFO,"X=%6d Y=%6d Z=%6d \r\n", data.AXIS_X, data.AXIS_Y, data.AXIS_Z);
						old_position = position;
				}else{
          err("LIS3DH: error while getting axis data");
        }
		}

    lis3dh_deinit();
    return obj;

}

```
