#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#include "toollib/bitmap.h"
#include "toollib/sapphire_image.h"
#include "toollib/palette.h"
#include "toollib/png_image.h"

#define MY_MIND

/* 定义 */
#define TRUE						(1)
#define FALSE						(0)

/* 资源颜色格式 */
#define RES_COLOR_FORMAT_INVALID	(0)
#define RES_COLOR_FORMAT_1BIT		(1)
#define RES_COLOR_FORMAT_2BIT		(2)
#define RES_COLOR_FORMAT_8BIT		(8)
#define RES_COLOR_FORMAT_RGB565		(RES_COLOR_FORMAT_ARGB1555 + 1)
#define RES_COLOR_FORMAT_ARGB1555	(16)
#define RES_COLOR_FORMAT_ARGB8888	(32)

/* 阿尔法通道所占位数 */
#define RES_ALPHA_TBL_MEM_4BIT      (0x00000000UL)
#define RES_ALPHA_TBL_MEM_8BIT      (0x00000001UL)
#define RES_ALPHA_TBL_CP            (0x00000002UL)
#define RES_ALPHA_TBL_NONE          (0x00000003UL)

/* 语言设置 */
/* MY_MIND:语言选项可以通过软件去自动识别，不用这样标识出来。 */
#define RES_LANG_NOR				(0)
#define RES_LANG_CHS				(1)
#define RES_LANG_ENG				(2)
#define RES_LANG_RUS				(3)
#define RES_LANG_ARB 				(4)

/* MY_MIND: 可以在配置项中设置，不一定要写下来。 */
/* need modify start */
/* 通用资源的个数 */
#define RES_NORMAL_IMAGE_COUNT		(450)
/* 通用资源的最后一个 */
#define RES_NORMAL_FINALLY_RES		(449)
/* 多语言资源的最后一个 */
#define RES_LANGUAGE_FINALLY_RES	(825)
/* need modify end */
/* 资源信息结构体 */
typedef struct
{
	int resid;
	char resenum[64];
	char path[260];
	int font;
	int left;
	int top;
	int width;
	int height;
	int transparent;
	unsigned int trans_color;
	unsigned int base;
	int format;
	char palette[64];
	char compression[64];
	char multi_language;
	int alphaSelect;
} resinfo_t;

/* need mofify start */
static unsigned char gu8FLASH_BIN[32 * 1024 * 1024]; // 32MB
/* need mofify end */

/*读取CSV-数字*/
const char * read_csv_int(const char *rowbuf, int *value)
{
	int pos = 0;
	const char *ptr = NULL;
	
	*value = 0;
	while(isdigit(rowbuf[pos]))
	{
		*value *= 10;
		*value += rowbuf[pos] - '0';
		
		pos++;
	}
	
	if(rowbuf[pos] == ',')
	{
		pos++;
		ptr = &rowbuf[pos];
	}
	
	if(rowbuf[pos] == '\0')
	{
		ptr = NULL;
	}
	
	return ptr;
}
/* 读取-字符串 */
const char * read_csv_str(const char *rowbuf, char *str)
{
	int pos = 0;
	const char *ptr = NULL;
	
	str[pos] = rowbuf[pos];
	while(rowbuf[pos] != ',')
	{
		str[pos] = rowbuf[pos];
		
		if(rowbuf[pos] == '\0')
			break;

		pos++;
	}
	
	if(rowbuf[pos] == ',')
	{
		str[pos] = '\0';
		pos++;
		ptr = &rowbuf[pos];
	}
	
	if(rowbuf[pos] == '\0')
	{
		str[pos] = '\0';
		ptr = NULL;
	}
	
	return ptr;
}
/* 读取CSV=十六进制 */
const char * read_csv_hex(const char *rowbuf, unsigned int *value)
{
	int pos = 0;
	const char *ptr = NULL;

	*value = 0;	
	if(rowbuf[0] != '0' || (rowbuf[1] != 'x' && rowbuf[1] != 'X'))
	{
		pos = 0;
	}
	else
	{
		pos = 2;
		while(isxdigit(rowbuf[pos]))
		{
			if(('0' <= rowbuf[pos]) && (rowbuf[pos] <= '9'))
			{
				*value *= 16;
				*value += rowbuf[pos] - '0';
			}
			else if(('a' <= rowbuf[pos]) && (rowbuf[pos] <= 'f'))
			{
				*value *= 16;
				*value += rowbuf[pos] - 'a' + 10;
			}
			else if(('A' <= rowbuf[pos]) && (rowbuf[pos] <= 'F'))
			{
				*value *= 16;
				*value += rowbuf[pos] - 'A' + 10;
			}
			
			pos++;
		}
	}
	
	if(rowbuf[pos] == ',')
	{
		pos++;
		ptr = &rowbuf[pos];
	}
	
	if(rowbuf[pos] == '\0')
	{
		ptr = NULL;
	}
	
	return ptr;
}

/* 加载资源信息 */
void load_resinfo(const char *rowbuf, resinfo_t *info)
{
	const char *ptr = rowbuf;
	
	char temp[256];
	
	do
	{
		// INIT INVALID VALUE
		/*  */
		info->resid = -1;
		info->resenum[0] = 0;
		info->path[0] = 0;
		info->font = 0;
		info->left = 0;
		info->top = 0;
		info->width = 0;
		info->height = 0;
		info->transparent = 0;
		info->trans_color = 0;
		info->base = 0;
		info->format = 0;
		info->palette[0] = 0;
		info->compression[0] = 0;
		info->multi_language = 0;
		
		// RES_ID
		ptr = read_csv_int(ptr, &info->resid);
		if(ptr == NULL) break;
		
		// RES_ENUM
		ptr = read_csv_str(ptr, info->resenum);
		if(ptr == NULL) break;
		
		// PATH
		ptr = read_csv_str(ptr, info->path);
		if(ptr == NULL) break;

		// FONT
		info->font = 0;
		ptr = read_csv_str(ptr, temp);
		if(ptr == NULL) break;
		if(strcmp(temp, "TRUE") == 0)
		{
			info->font = TRUE;
		}
		
		// LEFT
		ptr = read_csv_int(ptr, &info->left);
		if(ptr == NULL) break;
		
		// TOP
		ptr = read_csv_int(ptr, &info->top);
		if(ptr == NULL) break;
		
		// WIDTH
		ptr = read_csv_int(ptr, &info->width);
		if(ptr == NULL) break;
		
		// HEIGHT
		ptr = read_csv_int(ptr, &info->height);
		if(ptr == NULL) break;
		
		// TRANSPARENT
		info->transparent = 0;
		ptr = read_csv_str(ptr, temp);
		if(ptr == NULL) break;
		if(strcmp(temp, "TRUE") == 0)
		{
			info->transparent = 1;
		}
		
		// TRANSPARENT_COLOR
		ptr = read_csv_hex(ptr, &info->trans_color);
		if(ptr == NULL) break;
		
		// FLASH_BASE
		ptr = read_csv_hex(ptr, &info->base);
		if(ptr == NULL) break;
		
		// COLOR_FORMAT
		ptr = read_csv_str(ptr, temp);
		if(ptr == NULL) break;
		if(strcmp(temp, "RES_COLOR_FORMAT_1BIT") == 0)
		{
			info->format = RES_COLOR_FORMAT_1BIT;
		}
        else if(strcmp(temp, "RES_COLOR_FORMAT_2BIT") == 0)
        {
            info->format = RES_COLOR_FORMAT_2BIT;
        }
		else if(strcmp(temp, "RES_COLOR_FORMAT_8BIT") == 0)
		{
			info->format = RES_COLOR_FORMAT_8BIT;
		}
		else if(strcmp(temp, "RES_COLOR_FORMAT_ARGB1555") == 0)
		{
			info->format = RES_COLOR_FORMAT_ARGB1555;
		}
		else if(strcmp(temp, "RES_COLOR_FORMAT_ARGB565") == 0)
        {
            info->format = RES_COLOR_FORMAT_RGB565;
        }
		else if(strcmp(temp, "RES_COLOR_FORMAT_ARGB8888") == 0)
		{
			info->format = RES_COLOR_FORMAT_ARGB8888;
		}
		else
		{
			info->format = RES_COLOR_FORMAT_INVALID;
		}
				
		// PALETTE
		ptr = read_csv_str(ptr, info->palette);
		if(ptr == NULL) break;
		
		// COMPRESSION
		ptr = read_csv_str(ptr, info->compression);
		if(ptr == NULL) break;

		// REM
		ptr = read_csv_str(ptr, temp);
		if(ptr == NULL) break;
		
		// MULTI-LANGUAGE
		info->multi_language = RES_LANG_NOR;
		ptr = read_csv_str(ptr, temp);
		if(strcmp(temp, "CHS") == 0)
		{
			info->multi_language = RES_LANG_CHS;
		}
		else if(strcmp(temp, "ENG") == 0)
		{
			info->multi_language = RES_LANG_ENG;
		}
		else if(strcmp(temp, "RUS") == 0)
		{
			info->multi_language = RES_LANG_RUS;
		}
		else if(strcmp(temp, "ARB") == 0)
		{
			info->multi_language = RES_LANG_ARB;
		}
		else
		{}
		if(ptr == NULL) break;
		
		// alpha select
		info->alphaSelect = RES_ALPHA_TBL_NONE;
		ptr = read_csv_str(ptr, temp);
		if(strcmp(temp, "RES_ALPHA_TBL_MEM_4BIT") == 0)
		{
			info->alphaSelect = RES_ALPHA_TBL_MEM_4BIT;
		}
		else if (strcmp(temp, "RES_ALPHA_TBL_MEM_8BIT") == 0)
		{
			info->alphaSelect = RES_ALPHA_TBL_MEM_8BIT;
		}
		else if (strcmp(temp, "RES_ALPHA_TBL_CP") == 0)
		{
			info->alphaSelect = RES_ALPHA_TBL_CP;
		}
		else
		{
			info->alphaSelect = RES_ALPHA_TBL_NONE;
		}
		if(ptr == NULL) break;	
	} while(0);
}

void trim_bitmap(bitmap_t *bmp)
{
	int x, y;
	do
	{
		if(bmp == NULL)
			break;
		
		if(bmp->bitcount != 32)
			break;
		
		for(y = 0; y < bmp->height; y++)
		{
			for(x = 0; x < bmp->width; x++)
			{
                /* ͸����Ϊ0�Ĳ��֣�����Ϊ��ɫ */
				if(bmp->data[bmp->widthbyte * y + 4 * x + 3] == 0)
				{
					bmp->data[bmp->widthbyte * y + 4 * x + 0] = 0xFF;
					bmp->data[bmp->widthbyte * y + 4 * x + 1] = 0xFF;
					bmp->data[bmp->widthbyte * y + 4 * x + 2] = 0xFF;
				}
			}
		}

	} while(0);
}

bitmap_t * rect_and_rotate(const char *pathin, int left, int top, int width, int height)
{
	bitmap_t * in = NULL;
	bitmap_t * out = NULL;
	int x, y;
	
	int src, dest;
	
	int sx, sy;
	int dx, dy;
	
	int bytecount = 1;
	
	unsigned char mask1[8] = { 0x80, 0x40, 0x20, 0x10, 0x08, 0x04, 0x02, 0x01 };
    unsigned char mask2[4] = { 0xC0, 0x30, 0x0C, 0x03};
    unsigned char mask[4] = {0, 2, 4, 6};
	png_image_t *pngImage = NULL;
	do
	{
		
		if (strncmp(pathin + strlen(pathin) - 4, ".bmp", 4) == 0)
		{
			in = load_bitmap(pathin);
		}
		else if (strncmp(pathin + strlen(pathin) - 4, ".png", 4) == 0)
		{
			pngImage = load_png_image(pathin);
			in = png_image_to_bitmap(pngImage);
			trim_bitmap(in);
			free_png_image(pngImage);	
		}
		if(in == NULL)
			break;
		
		out = create_bitmap(height, width, in->bitcount, in->palette);
		if(out == NULL)
			break;

		if(in->bitcount >= 8)
		{
			// 32, 24, 16, 8 ==> 4, 3, 2, 1
			bytecount = in->bitcount / 8;
			
			for(y = 0; y < height; y++)
			{
				for(x = 0; x < width; x++)
				{
					// UI����ϵ 
					sx = left + x; sy = top + y;
					src = sy * in->widthbyte + sx * bytecount;
	
					// LCD����ϵ 
					dx = height - 1 - y; dy = x;				
					dest = dy * out->widthbyte + dx * bytecount;
					
					switch(in->bitcount)
					{
					case 32:
						out->data[dest + 3] = in->data[src + 3];
					case 24:
						out->data[dest + 2] = in->data[src + 2];
					case 16:
						out->data[dest + 1] = in->data[src + 1];
					case 8:
						out->data[dest] = in->data[src];
						break;
					}
				}
			}
		}
		else if(in->bitcount == 4)
		{
			
		}
        else if (in->bitcount == 2)
        {
            for (y = 0; y < height; y++)
            {
                for (x = 0; x < width; x++)
                {
                    //UI��?����?��
                    sx = left + x; sy = top + y;
                    src = sy * in->widthbyte + sx / 4;

                    //LCD��?����?��
                    dx = height - 1 - y; dy = x;
                    dest = dy * out->widthbyte + dx / 4;

                    out->data[dest] |= (((in->data[src] & mask2[sx % 4]) << mask[dy % 4]) >> mask[dx % 4]) ;
                }
            }
        }
		else if(in->bitcount == 1)
		{
			for(y = 0; y < height; y++)
			{
				for(x = 0; x < width; x++)
				{
					// UI����ϵ 
					sx = left + x; sy = top + y;
					src = sy * in->widthbyte + sx / 8;

					// LCD����ϵ 
					dx = height - 1 - y; dy = x;				
					dest = dy * out->widthbyte + dx / 8;
					
					if(in->data[src] & mask1[sx % 8])
					{
						out->data[dest] |= mask1[dx % 8];
					}
				}
			}
		}
		else
		{
			break;
		}

	} while(0);
	
	if(in) free_bitmap(in);

	return out;
}

void save_to_flash(unsigned char flash[], unsigned int offset, const bitmap_t *bmp)
{
	int src, bytecount, filesize;
	int x, y, width, height;
	
	sapphire_image_t * img = NULL;
	
	do
	{
		if(bmp == NULL)
			break;
			
		if(flash == NULL)
			break;
		
		if(bmp->bitcount >= 8)
		{
			// 32, 24, 16, 8 ==> 4, 3, 2, 1
			bytecount = bmp->bitcount / 8;
			/*
			if(bmp->bitcount == 16)
			{
				memcpy(&flash[offset], bitfieleds, 3 * 4L);
				offset += 3 * 4L;
			}
			*/
			for(y = 0; y < bmp->height; y++)
			{
				src = y * bmp->widthbyte;
				memcpy(&flash[offset], &bmp->data[src], bytecount * bmp->width);
				offset += bytecount * bmp->width;
			}
			
			//filesize = bytecount * bmp->width * bmp->height;
			//if(filesize % 4)
			//{
			//	fwrite("\0\0\0", 4 - (filesize % 4), 1L, file);
			//}
		}
		else if(bmp->bitcount == 4)
		{
			img = bitmap4bit_to_sapphire_image(bmp);
			if(img)
			{
				memcpy(&flash[offset], img->data, img->imagesize);
				offset += img->imagesize;
				free_sapphire_image(img);
				img = NULL;
			}
			else
			{
				printf("transform error, bitcount = 4, offset = 0x%08X\n", offset);
			}
		}
        else if (bmp->bitcount == 2)
        {
            img = bitmap2bit_to_sapphire_image(bmp);
            if (img)
            {
                memcpy(&flash[offset], img->data, img->imagesize);
                offset += img->imagesize;
                free_sapphire_image(img);
                img = NULL;
            }
        }
		else if(bmp->bitcount == 1)
		{
			img = bitmap1bit_to_sapphire_image(bmp);
			if(img)
			{
				memcpy(&flash[offset], img->data, img->imagesize);
				offset += img->imagesize;
				free_sapphire_image(img);
				img = NULL;
			}
			else
			{
				printf("transform error, bitcount = 1, offset = 0x%08X\n", offset);
			}
		}
		
	} while(0);
}


unsigned int checksum32(unsigned char flash[], unsigned int size)
{
	unsigned int chksum = 0;
	unsigned int i;
	
	for(i = 0; i < size; i++)
	{
		// little endian
		chksum += (unsigned int)flash[i];
	}
	
	return (chksum);
}

int output_flash_bin(const resinfo_t *info, unsigned int bin_offset)
{
	bitmap_t * bmp_step1 = NULL;
	bitmap_t * bmp_step2 = NULL;
	bitmap_t * bmp_step3 = NULL;

    bitmap_t * bmp_1bit = NULL;
    bitmap_t * bmp_4bit = NULL;

	char inpath[260];
	char outpath[260];
	
	int success = FALSE;
	char *postfix = NULL;
	do
	{
		if(info == NULL || bin_offset < 0x02400000)
			break;
	
		bin_offset -= 0x02400000;
	
		sprintf(inpath, "bitmap\\%s", info->path);
		switch (info->multi_language)
		{
		case RES_LANG_CHS:
			postfix = "_chs";
			break;
		case RES_LANG_ENG:
			postfix = "_eng";
			break;
		case RES_LANG_RUS:
			postfix = "_rus";
			break;
		case RES_LANG_ARB:
			postfix = "_arb";
			break;
		case RES_LANG_NOR:
			postfix = "_nor";
			break;
		default:
			postfix = "_error";
			break;
			
		}
		sprintf(outpath, "flash\\0x%08X\\%d%s.bmp", info->base, info->resid, postfix);
		
		// ��ȡ����תͼ�� 
		bmp_step1 = rect_and_rotate(inpath, info->left, info->top, info->width, info->height);
		if(bmp_step1 == NULL) 
		{
			printf("failed to rect_and_rotate %s\n", info->path);	
			break;
		}
		
		// ����ͼ��
		// ����ͼ��λ���ת������ֱ������ 
		if((bmp_step1->bitcount == info->format) && (info->format != RES_COLOR_FORMAT_RGB565))
		{
			save_bitmap(outpath, bmp_step1);
			save_to_flash(gu8FLASH_BIN, bin_offset, bmp_step1);
		}
		else if((bmp_step1->bitcount == 16) && (info->format == RES_COLOR_FORMAT_RGB565))
		{
			bmp_step2 = bmp16tobmp565(bmp_step1);
            if(bmp_step2 != NULL)
            {
                save_bitmap565(outpath, bmp_step2);
                save_to_flash(gu8FLASH_BIN, bin_offset, bmp_step2);

                free_bitmap(bmp_step2);
                bmp_step2 = NULL;
            }
            // ��������������ת��ʧ�� 
            else
            {
                save_bitmap(outpath, bmp_step1);
                printf("failed to transform %s from %d to %d\n", info->path, bmp_step1->bitcount, info->format);
            }
		}
		// ������24λת16(1555)λ����ô���Խ���ת�� 
		else if(((bmp_step1->bitcount == 24) || (bmp_step1->bitcount == 32)) && (info->format == RES_COLOR_FORMAT_ARGB1555))
		{
			if (bmp_step1->bitcount == 32)
			{
				bmp_step2 = bmp32tobmp24(bmp_step1);
			}
			else
			{
				bmp_step2 = bmp_step1;
			}
			bmp_step3 = bmp24tobmp1555(bmp_step2);

			if(bmp_step3 != NULL)
			{
				save_bitmap(outpath, bmp_step3);
				save_to_flash(gu8FLASH_BIN, bin_offset, bmp_step3);

				free_bitmap(bmp_step3);
				bmp_step3 = NULL;
			}
			// ��������������ת��ʧ�� 
			else
			{
				save_bitmap(outpath, bmp_step1);
				printf("failed to transform %s from %d to %d\n", info->path, bmp_step1->bitcount, info->format);
			}
		}
        // ������24λת16(565)λ����ô���Խ���ת�� 
        else if(((bmp_step1->bitcount == 24) || (bmp_step1->bitcount == 32)) && (info->format == RES_COLOR_FORMAT_RGB565))
        {
			if (bmp_step1->bitcount == 32)
			{
				bmp_step2 = bmp32tobmp24(bmp_step1);
			}
			else
			{
				bmp_step2 = bmp_step1;
			}
            bmp_step3 = bmp24tobmp565(bmp_step2);
            if(bmp_step3 != NULL)
            {
                save_bitmap565(outpath, bmp_step3);
                save_to_flash(gu8FLASH_BIN, bin_offset, bmp_step3);

                free_bitmap(bmp_step3);
                bmp_step3 = NULL;
            }
            // ��������������ת��ʧ�� 
            else
            {
                save_bitmap(outpath, bmp_step1);
                printf("failed to transform %s from %d to %d\n", info->path, bmp_step1->bitcount, info->format);
            }
        }
        /* ����5bitͼ����Դ */
        else if ((bmp_step1->bitcount == 32) && (info->alphaSelect == RES_ALPHA_TBL_MEM_4BIT) && (info->format == RES_COLOR_FORMAT_1BIT))
        {
            /* �ֱ�����1bit������Ϣ��4bit��alpha */
            bmp_4bit = bmp32tobmp4(bmp_step1);
            if (bmp_4bit != NULL)
            { 
                save_bitmap(outpath, bmp_4bit);
                /* ���¼���ƫ����*/
                bin_offset = bin_offset + (bmp_step1->height * bmp_step1->width * 1 + 31) / 32 * 4;
                /* �洢4bit��alpha��Ϣ */
                save_to_flash(gu8FLASH_BIN, bin_offset, bmp_4bit);
            } 
            else
            {
                save_bitmap(outpath, bmp_step1);
                printf("failed to transform %s from %d to %d\n", info->path, bmp_step1->bitcount, info->format);
            }
            free_bitmap(bmp_1bit);
            bmp_1bit = NULL;
            free_bitmap(bmp_4bit);
            bmp_4bit = NULL;
        }
		// ��������������ת��ʧ�� 
		else
		{
			save_bitmap(outpath, bmp_step1);
			printf("failed to transform %s from %d to %d\n", info->path, bmp_step1->bitcount, info->format);
		}
		
		free_bitmap(bmp_step1);
		bmp_step1 = NULL;
		
		success = TRUE;
	} while(0);
	
	return success;
}

int main(int argc, char *argv[])
{
	int i, len;
	char csvbuf[1024];
	char outpath[260];
	unsigned int chksum;
	
	resinfo_t info = { 0 };

	FILE *fileBitmapCSV = NULL;
	FILE *fileFlashCSV = NULL;
	FILE *fileFlashBIN = NULL;
	FILE *fileFlashH = NULL;
	FILE *fileFlashC = NULL;
	
	int lcd_left, lcd_top, lcd_width, lcd_height, lcd_bitcount, lcd_imagesize, bin_offset;
	const char *lcd_format;
	int lcd_base0x02400000 = 0x02400000;
	int lcd_base0x02420020 = 0x02420020;
	int lcd_base0x02463BE0 = 0x02463BE0;
	int lcd_base0x02C40000 = 0x02C40000;
	int lcd_base0x03040000 = 0x03040000;
	int lcd_base0x03300000 = 0x03300000;
	int lcd_base0x03400000 = 0x03400000;

	char *alphaSelect = NULL;
	
	do
	{
		fileBitmapCSV = fopen("bitmap\\bitmap.csv", "r");
		if(fileBitmapCSV == NULL)
		{
			printf("open bitmap\\bitmap.csv failed\n");
			break;
		}
		
		fileFlashCSV = fopen("flash\\flash.csv", "w");
		if(fileFlashCSV == NULL)
		{
			printf("open flash.csv failed\n");
			break;
		}
		
		fileFlashH = fopen("flash\\flash.h", "w");
		if(fileFlashH == NULL)
		{
			printf("open flash.h failed\n");
			break;
		}
		
		fileFlashC = fopen("flash\\flash.c", "w");
		if(fileFlashC == NULL)
		{
			printf("open flash.c failed\n");
			break;
		}

		fileFlashBIN = fopen("flash\\flash.bin", "wb");
		if(fileFlashBIN == NULL)
		{
			printf("open flash.bin failed\n");
			break;
		}
		
		// FLASH��ʼȫ����Ϊ0xFF���ͻ�Ҫ����
		memset(gu8FLASH_BIN, 0xFF, sizeof(gu8FLASH_BIN));
		
		// ���˵���һ�б�ͷ 
		fgets(csvbuf, 1024, fileBitmapCSV);
		fprintf(fileFlashCSV, "RES_ID,RES_ENUM,PATH,LEFT,TOP,WIDTH,HEIGHT,COLOR_FORMAT,ALPHASELECT,PALETTE,FLASH_ADDR,FLASH_SIZE\n");
		fprintf(fileFlashH, "typedef enum\n{\n    RES_ID_INVALID = -1,\n");
		fprintf(fileFlashC, "const RES_IMAGE_INFO_t_ gs_atImageInfoTable[RES_ID_NORMAL_IMAGE_COUNT] = {\n");
		
		// ���δ���ÿһ�� 
		while(fgets(csvbuf, 1024, fileBitmapCSV))
		{
			// ȥ����β'\n'
			len = strlen(csvbuf); 
			csvbuf[len - 1] = '\0';
			
			// ����CSV�ļ�����ȡ��Ҫ����Ϣ 
			load_resinfo(csvbuf, &info);
			printf("%d, %s\n", info.resid, info.path);

			do
			{
				// ͼ���ĸ߶ȣ�UI����ϵ���������ĵı���
				// ͼ���Ŀ��ȣ�LCD����ϵ���������ĵı��� 
				if(info.height % 4 != 0)
				{
					printf("ERROR:rect height must be multiple of four\n"); 
					system("PAUSE");
					goto EXIT_MAIN;
				} 
				
				// �����ǿ��У���ֱ���������� 
				if(info.path[0] == 0)
				{
					// д������ 
					fprintf(fileFlashCSV, "%d,RES_INVALID,,,,,,,,,,\n", info.resid);
					if ((info.resid == RES_NORMAL_FINALLY_RES) || (info.resid == RES_LANGUAGE_FINALLY_RES))
					{
						fprintf(fileFlashC, "    {gs_atImageLoadedInfoTable+%3d, 0x02400000, 0x00000000, 0x00000000, 0x00FF00FF,   0,   0, 0, RES_RLD_OFF, RES_COLOR_FORMAT_ARGB1555, RES_ALPHA_TBL_NONE    , RES_ID_PALETTE_NONE}\n", info.resid);
					}
					else
					{
						fprintf(fileFlashC, "    {gs_atImageLoadedInfoTable+%3d, 0x02400000, 0x00000000, 0x00000000, 0x00FF00FF,   0,   0, 0, RES_RLD_OFF, RES_COLOR_FORMAT_ARGB1555, RES_ALPHA_TBL_NONE    , RES_ID_PALETTE_NONE},\n", info.resid);
					}					

				}
				else
				{
					// ���õ�ǰ�ļ� 
					//sprintf(inpath, "bitmap\\%s", info.path);
					sprintf(outpath, "flash\\0x%08X\\%d.bmp", info.base, info.resid);
					
					// ��תͼƬ���� 
					if(info.font)
					{
						// ����ͼƬ�Ƕ���ͼƬ��װ��һ��ͼƬ����ô����ֻҪ������ת 
						lcd_left = 0;
						lcd_top = 0;
						lcd_width = info.height;
						lcd_height = info.width;
					}
					else
					{
						// ������ͨͼƬ����������UI����ϵ��ת��LCD����ϵ 
						lcd_left = 480 - info.top - info.height;
						lcd_top = info.left;
						lcd_width = info.height;
						lcd_height = info.width;
					}
					
					// ����ͼ����С 
					if (info.format == RES_COLOR_FORMAT_RGB565)
					{
						lcd_bitcount = 16;
					}
					else if (info.format ==  RES_COLOR_FORMAT_1BIT)
					{
						if (info.alphaSelect == RES_ALPHA_TBL_MEM_4BIT)
						{
							lcd_bitcount = 5;
						}
						else
						{
							lcd_bitcount = 1;
						}
					}
					else
					{
						lcd_bitcount = info.format;
					}
					lcd_imagesize = sapphire_imagesize(lcd_width, lcd_height, lcd_bitcount);
					
					// ����ͼ��ƫ���� 
					switch(info.base)
					{
					case 0x02420020:
						bin_offset = lcd_base0x02420020;
						lcd_base0x02420020 += lcd_imagesize;
						if (lcd_base0x02420020 > 0x02463BE0)
						{
							printf("ERROR:image range overflow\n"); 
							system("PAUSE");
							goto EXIT_MAIN;
						}
						break;
					case 0x02463BE0:
						bin_offset = lcd_base0x02463BE0;
						lcd_base0x02463BE0 += lcd_imagesize;
						if(lcd_base0x02463BE0 > 0x02C40000)
						{
							printf("ERROR:image range overflow\n"); 
							system("PAUSE");
							goto EXIT_MAIN;
						}
						break;
					case 0x02C40000:
						bin_offset = lcd_base0x02C40000;
						lcd_base0x02C40000 += lcd_imagesize;
						if(lcd_base0x02C40000 > 0x03040000)
						{
							printf("ERROR:image range overflow\n"); 
							system("PAUSE");
							goto EXIT_MAIN;
						}
						break;
					case 0x03040000:
						bin_offset = lcd_base0x03040000;
						lcd_base0x03040000 += lcd_imagesize;
						if(lcd_base0x03040000 > 0x03300000)
						{
							printf("ERROR:image range overflow\n"); 
							system("PAUSE");
							goto EXIT_MAIN;
						}
						break;
					case 0x03300000:
						bin_offset = lcd_base0x03300000;
						lcd_base0x03300000 += lcd_imagesize;
						if(lcd_base0x03300000 > 0x03400000)
						{
							printf("ERROR:image range overflow\n"); 
							system("PAUSE");
							goto EXIT_MAIN;
						}
						break;
					case 0x03400000:
						bin_offset = lcd_base0x03400000;
						lcd_base0x03400000 += lcd_imagesize;
						if(lcd_base0x03400000 > 0x03C00000)
						{
							printf("ERROR:image range overflow\n"); 
							system("PAUSE");
							goto EXIT_MAIN;
						}
						break;
					default:
						bin_offset = 0;
						printf("ERROR:image base must be 0x02420020 or 0x02463BE0 or 0x02C40000 or 0x03300000 or 0x03400000\n"); 
						system("PAUSE");
						goto EXIT_MAIN;
					}
					
					// ����ͼ�����ظ�ʽ 
					switch(lcd_bitcount)
					{
					case 1:
						lcd_format = "RES_COLOR_FORMAT_1BIT    ";
						break;
                    case 2:
                        lcd_format = "RES_COLOR_FORMAT_2BIT";
                        break;
					case 4:
						lcd_format = "RES_COLOR_FORMAT_4BIT    ";
						break;
					case 5:
						lcd_format = "RES_COLOR_FORMAT_1BIT    ";
						break;
					case 8:
						lcd_format = "RES_COLOR_FORMAT_8BIT    ";
						break;
					case 16:
						if(info.format == RES_COLOR_FORMAT_ARGB1555)
						{
							lcd_format = "RES_COLOR_FORMAT_ARGB1555";
						}
						else
						{
							lcd_format = "RES_COLOR_FORMAT_RGB565  ";
						}
						break;
					case 24:
						lcd_format = "RES_COLOR_FORMAT_RGB888  ";
						break;
					case 32:
						lcd_format = "RES_COLOR_FORMAT_ARGB8888";
						break;
					default:
						lcd_format = "RES_COLOR_FORMAT_INVALID ";
						printf("ERROR:unsupport color format\n"); 
						system("PAUSE");
						goto EXIT_MAIN;
					}

					switch(info.alphaSelect)
					{
					case RES_ALPHA_TBL_MEM_4BIT:
						alphaSelect = "RES_ALPHA_TBL_MEM_4BIT";
						break;
					case RES_ALPHA_TBL_MEM_8BIT:
						alphaSelect = "RES_ALPHA_TBL_MEM_8BIT";
						break;
					case RES_ALPHA_TBL_CP:
						alphaSelect = "RES_ALPHA_TBL_CP      ";
						break;
					case RES_ALPHA_TBL_NONE:
					default:
						alphaSelect = "RES_ALPHA_TBL_NONE    ";
						break;
					}
					
					fprintf(fileFlashCSV, "%d,%s,%s,%d,%d,%d,%d,%s,%s,%s,0x%08X,0x%08X\n", info.resid, info.resenum, outpath, lcd_left, lcd_top, lcd_width, lcd_height, lcd_format, alphaSelect, info.palette, bin_offset, lcd_imagesize);
					
					/* add by taow 2014-12-29 begin */
					if ((info.resid == RES_NORMAL_IMAGE_COUNT) && (info.multi_language == RES_LANG_CHS))
					{
					    fprintf(fileFlashH, "    RES_ID_NORMAL_IMAGE_COUNT = %d,\n", info.resid);
					}
					/* add by taow 2014-12-29 end */
					if ((info.multi_language == RES_LANG_CHS) || (info.multi_language == RES_LANG_NOR))
					{
						fprintf(fileFlashH, "    %s = %d,\n", info.resenum, info.resid);
					}
					
					if ((info.resid == RES_NORMAL_IMAGE_COUNT) && (info.multi_language == RES_LANG_CHS))
					{
						fprintf(fileFlashC, "};\n");
						fprintf(fileFlashC, "const RES_IMAGE_INFO_t_ gs_atImageInfoChnTable[RES_ID_IMAGE_COUNT-RES_ID_NORMAL_IMAGE_COUNT] = {\n");
					}
					else if ((info.resid == RES_NORMAL_IMAGE_COUNT) && (info.multi_language == RES_LANG_ENG))
					{
						fprintf(fileFlashC, "};\n");
						fprintf(fileFlashC, "const RES_IMAGE_INFO_t_ gs_atImageInfoEngTable[RES_ID_IMAGE_COUNT-RES_ID_NORMAL_IMAGE_COUNT] = {\n");
					}
					else if ((info.resid == RES_NORMAL_IMAGE_COUNT) && (info.multi_language == RES_LANG_RUS))
					{
						fprintf(fileFlashC, "};\n");
						fprintf(fileFlashC, "const RES_IMAGE_INFO_t_ gs_atImageInfoRusTable[RES_ID_IMAGE_COUNT-RES_ID_NORMAL_IMAGE_COUNT] = {\n");
					}
					else if ((info.resid == RES_NORMAL_IMAGE_COUNT) && (info.multi_language == RES_LANG_ARB))
					{
						fprintf(fileFlashC, "};\n");
						fprintf(fileFlashC, "const RES_IMAGE_INFO_t_ gs_atImageInfoArbTable[RES_ID_IMAGE_COUNT-RES_ID_NORMAL_IMAGE_COUNT] = {\n");
					}
					else
					{}

					fprintf(fileFlashC, "    {gs_atImageLoadedInfoTable+%3d, 0x%08X, 0x%08X, 0x%08X, 0x%08X, %3d, %3d, %d, %s, %s, %s, %s}", 
					    info.resid, bin_offset, lcd_imagesize, lcd_imagesize, info.trans_color, lcd_width, lcd_height, info.transparent, info.compression, lcd_format, alphaSelect,  info.palette);

					if ((info.resid == RES_NORMAL_FINALLY_RES) || (info.resid == RES_LANGUAGE_FINALLY_RES))
					{
						fprintf(fileFlashC, "  /* %s */\n", info.path);
					}
					else
					{
						fprintf(fileFlashC, ", /* %s */\n", info.path);
					}

					if(!output_flash_bin(&info, bin_offset))
					{
						printf("ERROR:output_flash_bin failed\n"); 
						system("PAUSE");
						goto EXIT_MAIN;
					}
				}
			} while(0);
		}
		
		fprintf(fileFlashH, "    RES_ID_IMAGE_COUNT = %d\n} HMI_RESOURCE_ID;\n", RES_LANGUAGE_FINALLY_RES+1);
		fprintf(fileFlashC, "};\n");

		// У���� 
		chksum = checksum32(gu8FLASH_BIN, sizeof(gu8FLASH_BIN) - 4L);
		chksum = (unsigned int)(~chksum);
		gu8FLASH_BIN[sizeof(gu8FLASH_BIN) - 1L] = (unsigned char)(chksum & 0xFFu);
		gu8FLASH_BIN[sizeof(gu8FLASH_BIN) - 2L] = (unsigned char)((chksum >> 8) & 0xFFu);
		gu8FLASH_BIN[sizeof(gu8FLASH_BIN) - 3L] = (unsigned char)((chksum >> 16) & 0xFFu);
		gu8FLASH_BIN[sizeof(gu8FLASH_BIN) - 4L] = (unsigned char)(chksum >> 24);		
		// д�뵽�ļ� 
		fwrite(gu8FLASH_BIN, sizeof(gu8FLASH_BIN), 1L, fileFlashBIN);
		
	} while(0);

EXIT_MAIN:

	if(fileBitmapCSV) fclose(fileBitmapCSV);
	if(fileFlashCSV) fclose(fileFlashCSV);
	if(fileFlashBIN) fclose(fileFlashBIN);
	if(fileFlashC) fclose(fileFlashC);
	if(fileFlashH) fclose(fileFlashH);

	return 0;
}

