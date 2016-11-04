include $(TOPDIR)/rules.mk

PKG_NAME:=aws-iot-prj
PKG_REV:=1
PKG_VERSION:=$(PKG_REV)
PKG_RELEASE:=1

include $(INCLUDE_DIR)/package.mk

define Package/aws-iot-prj
  SECTION:=net
  CATEGORY:=Network
  DEPENDS:=openssl
  TITLE:=aws-iot-prj
endef

define Package/iot-prj/description
	These files are the iot-prj files to support aws-iot over openwrt
endef

#ENABLE_DASHUI:=$(if $(ENABLE_DASHUI),$(ENABLE_DASHUI),0)

define Build/Prepare
#	# Copy dummy makefile into the build dir to keep the openwrt build system happy
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Build/Compile
	APP_DIR = $(PKG_BUILD_DIR)/iot-prj
	APP_INCLUDE_DIRS += -I $(APP_DIR)
	APP_NAME=iot-prj
	APP_SRC_FILES=$(APP_NAME).c

	#IoT client directory
	IOT_CLIENT_DIR= $(PKG_BUILD_DIR)/aws_iot_src
	IOT_INCLUDE_DIRS += -I $(IOT_CLIENT_DIR)/protocol/mqtt
	IOT_INCLUDE_DIRS += -I $(IOT_CLIENT_DIR)/protocol/mqtt/aws_iot_embedded_client_wrapper
	IOT_INCLUDE_DIRS += -I $(IOT_CLIENT_DIR)/protocol/mqtt/aws_iot_embedded_client_wrapper/platform_linux
	IOT_INCLUDE_DIRS += -I $(IOT_CLIENT_DIR)/protocol/mqtt/aws_iot_embedded_client_wrapper/platform_linux/common
	IOT_INCLUDE_DIRS += -I $(IOT_CLIENT_DIR)/protocol/mqtt/aws_iot_embedded_client_wrapper/platform_linux/openssl
	IOT_INCLUDE_DIRS += -I $(IOT_CLIENT_DIR)/utils

	PLATFORM_DIR = $(IOT_CLIENT_DIR)/protocol/mqtt/aws_iot_embedded_client_wrapper/platform_linux/openssl
	PLATFORM_COMMON_DIR = $(IOT_CLIENT_DIR)/protocol/mqtt/aws_iot_embedded_client_wrapper/platform_linux/common
	IOT_SRC_FILES += $(IOT_CLIENT_DIR)/protocol/mqtt/aws_iot_embedded_client_wrapper/aws_iot_mqtt_embedded_client_wrapper.c
	IOT_SRC_FILES += $(shell find $(PLATFORM_DIR)/ -name '*.c')
	IOT_SRC_FILES += $(shell find $(PLATFORM_COMMON_DIR)/ -name '*.c')

#MQTT Paho Embedded C client directory
	MQTT_DIR = $(PKG_BUILD_DIR)/aws_mqtt_embedded_client_lib
	MQTT_C_DIR = $(MQTT_DIR)/MQTTClient-C/src
	MQTT_EMB_DIR = $(MQTT_DIR)/MQTTPacket/src

	MQTT_INCLUDE_DIR += -I $(MQTT_EMB_DIR)
	MQTT_INCLUDE_DIR += -I $(MQTT_C_DIR)

	MQTT_SRC_FILES += $(shell find $(MQTT_EMB_DIR)/ -name '*.c')
	MQTT_SRC_FILES += $(MQTT_C_DIR)/MQTTClient.c


#TLS - openSSL
	TLS_LIB_DIR = $(STAGING_DIR)/usr/lib/
	TLS_INCLUDE_DIR = -I $(STAGING_DIR)/usr/include/openssl
	EXTERNAL_LIBS += -L$(TLS_LIB_DIR)
	LD_FLAG := -ldl -lssl -lcrypto
	LD_FLAG += -Wl,-rpath,$(TLS_LIB_DIR)

#Aggregate all include and src directories
	INCLUDE_ALL_DIRS += $(IOT_INCLUDE_DIRS) 
	INCLUDE_ALL_DIRS += $(MQTT_INCLUDE_DIR) 
	INCLUDE_ALL_DIRS += $(TLS_INCLUDE_DIR)
	INCLUDE_ALL_DIRS += $(APP_INCLUDE_DIRS)
 
	
	SRC_FILES += $(MQTT_SRC_FILES)
	SRC_FILES += $(APP_SRC_FILES)
	SRC_FILES += $(IOT_SRC_FILES)

# Logging level control
	LOG_FLAGS += -DIOT_DEBUG
	LOG_FLAGS += -DIOT_INFO
	LOG_FLAGS += -DIOT_WARN
	LOG_FLAGS += -DIOT_ERROR


	COMPILER_FLAGS += -g
	COMPILER_FLAGS += $(LOG_FLAGS)
#If the processor is big endian uncomment the compiler flag
	#COMPILER_FLAGS += -DREVERSED

	

	$(MAKE) -C $(PKG_BUILD_DIR) $(SRC_FILES) \
	C_INCLUDE_PATH="$(STAGING_DIR)/usr/include" \
	CPLUS_INCLUDE_PATH="$(STAGING_DIR)/usr/include" \
	CC="$(TARGET_CC)" \
	EXTRA_CFLAGS="$(TARGET_CFLAGS) $(COMPILER_FLAGS)"\
	CROSS_COMPILE="$(TARGET_CROSS)" \
	CPPFLAGS="-I$(STAGING_DIR)/usr/include \
	 -I$(STAGING_DIR)/usr/include/uClibc++ -fno-builtin -fno-rtti \
	 -nostdinc++ -fpermissive -Wno-error"  $(INCLUDE_ALL_DIRS) \
	LDFLAGS="$(TARGET_LDFLAGS) $(LD_FLAG) $(EXTERNAL_LIBS) "
	

#MAKE_CMD = $(CC) $(SRC_FILES) $(COMPILER_FLAGS) -o $(APP_NAME) $(LD_FLAG) $(EXTERNAL_LIBS) $(INCLUDE_ALL_DIRS)

endef

#define Package/ozwd/config
#	config OZWD_USE_RESET_GPIO
#		bool "Use shell script to reset z-wave controller"
#		default y
#	config OZWD_ZWAVE_RESET_GPIO
#		depends OZWD_USE_RESET_GPIO
#		help
#			The GPIO used to reset z-wave. On the NA900 this should be 27, for NA301 and NA930 it should be 61 .
#		int "ZWave Reset GPIO"
#		default 61
#endef

define Package/aws-iot-prj/install
	$(INSTALL_DIR) $(1)/usr/iot-prj
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/aws-iot-prj $(1)/usr/sbin
	#git describe --long --all --dirty > $(1)/etc/ozwd/version || echo NOT_VERSIONED > $(1)/etc/ozwd/version
endef

$(eval $(call BuildPackage,iot-prj))

