# PX4
飞行控制

飞控的crc检验用的
CRC16_CCITT_FALSE：多项式x16+x12+x5+1（0x1021），初始值0xFFFF，低位在后，高位在前，结果与0x0000异或
飞控校验代码：
/**
 * @brief Accumulate the X.25 CRC by adding one char at a time.
 *
 * The checksum function adds the hash of one char at a time to the
 * 16 bit checksum (uint16_t).
 *
 * @param data new char to hash
 * @param crcAccum the already accumulated checksum
 **/
static inline void crc_accumulate(uint8_t data, uint16_t *crcAccum)
{
        /*Accumulate one byte of data into the CRC*/
        uint8_t tmp;

        tmp = data ^ (uint8_t)(*crcAccum &0xff);
        tmp ^= (tmp<<4);
        *crcAccum = (*crcAccum>>8) ^ (tmp<<8) ^ (tmp <<3) ^ (tmp>>4);
}
#endif


/**
 * @brief Initiliaze the buffer for the X.25 CRC
 *
 * @param crcAccum the 16 bit X.25 CRC
 */
static inline void crc_init(uint16_t* crcAccum)
{
        *crcAccum = X25_INIT_CRC;
}


/**
 * @brief Calculates the X.25 checksum on a byte buffer
 *
 * @param  pBuffer buffer containing the byte array to hash
 * @param  length  length of the byte array
 * @return the checksum over the buffer bytes
 **/
static inline uint16_t crc_calculate(const uint8_t* pBuffer, uint16_t length)
{
        uint16_t crcTmp;
        crc_init(&crcTmp);
	while (length--) {
                crc_accumulate(*pBuffer++, &crcTmp);
        }
        return crcTmp;
}


/**
 * @brief Accumulate the X.25 CRC by adding an array of bytes
 *
 * The checksum function adds the hash of one char at a time to the
 * 16 bit checksum (uint16_t).
 *
 * @param data new bytes to hash
 * @param crcAccum the already accumulated checksum
 **/
static inline void crc_accumulate_buffer(uint16_t *crcAccum, const char *pBuffer, uint16_t length)
{
	const uint8_t *p = (const uint8_t *)pBuffer;
	while (length--) {
                crc_accumulate(*p++, crcAccum);
        }
}



电信协议校验
CRC16_CCITT：多项式x16+x12+x5+1（0x1021），初始值0x0000，低位在前，高位在后，结果与0x0000异或
这个校验代码：
void InvertUint8(unsigned char *dBuf,unsigned char *srcBuf)
{
	int i;
	unsigned char tmp[4];
	tmp[0] = 0;
	for (i=0; i< 8; i++) {
	if (srcBuf[0]& (1 << i))
		tmp[0]|=1<<(7-i);

	}
	dBuf[0] = tmp[0];
}

void InvertUint16(unsigned short *dBuf,unsigned short *srcBuf)  
{
	int i;
	unsigned short tmp[4];
	tmp[0] = 0;
	for (i=0; i< 16; i++) 
	{
	if (srcBuf[0]& (1 << i))
	tmp[0]|=1<<(15 - i);
	}
	dBuf[0] = tmp[0];
}

unsigned short CRC16_CCITT(unsigned char *puchMsg, unsigned int usDataLen)
{  
	unsigned short wCRCin = 0x0000; 
	unsigned short wCPoly = 0x1021; 
	unsigned char wChar = 0;   
	int i = 0;
	while (usDataLen--) 	  
	{       
		wChar = *(puchMsg++);        
		InvertUint8(&wChar,&wChar);        
		wCRCin ^= (wChar << 8);       
		for(i = 0;i < 8;i++)        
		{          
		if(wCRCin & 0x8000)            
		wCRCin = (wCRCin << 1) ^ wCPoly;         
		else            
		wCRCin = wCRCin << 1;        
		}  
	}  
	InvertUint16(&wCRCin,&wCRCin); 
	return (wCRCin) ;
 }







