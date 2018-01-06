# Implementation in Code:

```c
#define             BME280_SPI_CS_PIN                         2
#define             BME280_SPI_SDI_PIN                        3
#define             BME280_SPI_SCK_PIN                        4
#define             BME280_SPI_SDO_PIN                        5
#define             I2C_TIMEOUT                               100000


uint8_t   SPI_Tx_Buf[SPI_BUFSIZE];
uint8_t   SPI_Rx_Buf[SPI_BUFSIZE];
volatile  uint8_t   SPIReadLength, SPIWriteLength;
bool BME280_INIT = false;
static volatile bool spi_xfer_done;  
static const nrf_drv_spi_t spi = NRF_DRV_SPI_INSTANCE(SPI_INSTANCE);  

void spi_event_handler(nrf_drv_spi_evt_t const * p_event)
{
    spi_xfer_done = true;
}

static uint32_t bme280_spi_init(void)
{
    uint32_t err_code;
    nrf_drv_spi_config_t spi_bme_config = NRF_DRV_SPI_DEFAULT_CONFIG;
    spi_bme_config.ss_pin   = BME280_SPI_CS_PIN;
    spi_bme_config.miso_pin = BME280_SPI_SDO_PIN;
    spi_bme_config.mosi_pin = BME280_SPI_SDI_PIN;
    spi_bme_config.sck_pin  = BME280_SPI_SCK_PIN;

    //set SPI transfer mode as blocking
    if(!BME280_INIT) err_code = nrf_drv_spi_init(&spi, &spi_bme_config, spi_event_handler);
    if(err_code != NRF_SUCCESS)
	  {
        err("BME280: Error while SPI Init");
        return err_code;
	  }else{
      BME280_INIT = TRUE;
    }

	  return NRF_SUCCESS;
}

void user_delay_ms(uint32_t period)
{
  		nrf_delay_ms(period);
}

int8_t user_spi_read(uint8_t dev_id, uint8_t reg_addr, uint8_t *reg_data, uint16_t len)
{
    int8_t rslt = 0;

    spi_xfer_done = false;
		nrf_gpio_pin_write ( BME280_SPI_CS_PIN, 0 );

    SPI_Tx_Buf[0] = reg_addr;
    APP_ERROR_CHECK(nrf_drv_spi_transfer(&spi, SPI_Tx_Buf, 1, SPI_Rx_Buf, len+1));
	  while(spi_xfer_done == false);

    memcpy(reg_data, &SPI_Rx_Buf[1], len);
		nrf_gpio_pin_write ( BME280_SPI_CS_PIN, 1 );
    return rslt;
}

int8_t user_spi_write(uint8_t dev_id, uint8_t reg_addr, uint8_t *reg_data, uint16_t len)
{
    int8_t rslt = 0;
    spi_xfer_done = false;

    SPI_Tx_Buf[0] = reg_addr;
		memcpy(&SPI_Tx_Buf[1], reg_data, len);
		nrf_gpio_pin_write ( BME280_SPI_CS_PIN, 0 );
    APP_ERROR_CHECK(nrf_drv_spi_transfer(&spi, SPI_Tx_Buf, len+1, NULL, 0));
	  while(spi_xfer_done == false);
		nrf_gpio_pin_write ( BME280_SPI_CS_PIN, 1 );
    return rslt;
}


void itracker_bme280data(){
  struct bme280_dev dev;
  int8_t rslt = BME280_OK;

  bme280_spi_init();

  dev.dev_id = 0;
  dev.intf = BME280_SPI_INTF;
  dev.read = user_spi_read;
  dev.write = user_spi_write;
  dev.delay_ms = user_delay_ms;

  rslt = bme280_init(&dev);
  if(rslt != BME280_OK) {
    err("BME280 problem during device init");
  }

  	uint8_t settings_sel;
  	struct bme280_data comp_data;


  	dev.settings.osr_h = BME280_OVERSAMPLING_1X;
  	dev.settings.osr_p = BME280_OVERSAMPLING_16X;
  	dev.settings.osr_t = BME280_OVERSAMPLING_2X;
  	dev.settings.filter = BME280_FILTER_COEFF_16;
  	dev.settings.standby_time = BME280_STANDBY_TIME_62_5_MS;

  	settings_sel = BME280_OSR_PRESS_SEL;
  	settings_sel |= BME280_OSR_TEMP_SEL;
  	settings_sel |= BME280_OSR_HUM_SEL;
  	settings_sel |= BME280_STANDBY_SEL;
  	settings_sel |= BME280_FILTER_SEL;
  	rslt = bme280_set_sensor_settings(settings_sel, &dev);

    if(rslt != BME280_OK){
      err("BME280: issue while applying sensor settings");
    }

  	rslt = bme280_set_sensor_mode(BME280_NORMAL_MODE, &dev);

    if(rslt != BME280_OK){
      err("BME280: issue while setting sensor mode");
    }

  	rslt = bme280_get_sensor_data(BME280_ALL, &comp_data, &dev);

    if(rslt != BME280_OK){
      err("BME280: issue while getting sensor data");
    }
}

```
