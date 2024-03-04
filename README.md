# USB_Protocol

## Building

This project can be build like any other Cmake project from the root directory

    mkdir build && cd build && cmake .. && make && sudo make install

## Linking

Building will create a dynamic library and install to /usr/local/lib as well as make the protocol's public headers available in /usr/local/include. You can use protocol functions by including with #include <mechatronics_usb_protocol/mech_usb_protocol.h>

## Getting Started

`actions` - This is where the user can set which functions will be called for the associated topic/subtopic

`subtopics` - This will contain actual implementation of subsystems 

`parser` - The parser will take in a message (presumably sent over USB) and call the appropriate function

## Example CMake

    include_directories(
        src/utils/include
        src/main/include
        src/protocol/include
        src/protocol/include/parser
        src/protocol/include/subtopics
        src/protocol/include/actions
        src/protocol/util
    )

    set(USB_PROTOCOL_SOURCES 
        src/protocol/src/parser/usb_parse.c
        src/protocol/src/parser/usb_flags.c
        src/protocol/src/actions/usb_actions.c
    )

    set(USB_PROTOCOL_HEADERS 
        src/protocol/include/mech_usb_protocol.h
        src/protocol/include/parser/usb_parse.h
        src/protocol/include/parser/usb_flags.h
        src/protocol/include/parser/msg_types.h
        src/protocol/include/actions/usb_actions.h
        src/protocol/include/subtopics/system.h
        src/protocol/include/subtopics/power.h
        src/protocol/include/subtopics/motor.h
        src/protocol/util/byte.h
    )

    add_library(mechatronics_usb_protocol STATIC ${USB_PROTOCOL_SOURCES} ${USB_PROTOCOL_HEADERS})
    target_link_libraries(${PROJECT_NAME} mechatronics_usb_protocol)

## Using the protocol in embedded project

To use the protocol, in your main function you'll have to call a setup function

    init_robot_actions();

Then you can set specific implementations like this

    set_robot_actions(MOTORS, MOVE_MOTORS, &MOVE_MOTOR_FUNC); <--You'll have to define MOVE_MOTOR_FUNC

You'll need something in your main loop that does something to this effect

    ```
    for (;;)
    {
        const byte_t init = read_byte();
        if (init == INIT_BYTE)
        {
            const byte_t meta_flags_byte = read_byte();
            const MetaFlags meta_flags = EXTRACT_META_FLAGS(meta_flags_byte);
            const byte_t msg_size = meta_flags.MSG_SIZE - 2;
            byte_t buffer[msg_size] = {0};
            read_buffer(buffer, msg_size);
            HANDLE_MESSAGE(buffer);
        }   
    }
    ```

Here's an example for the SAM4E

    ```
    #include "mech_usb_protocol.h"

    #define USB_PACKET_SIZE_BYTES 32
    static unsigned char buffer[USB_PACKET_SIZE_BYTES] = {0};

    void enter_loop(void)
    {
        for (;;)
        {
            const int byte = udi_cdc_getc();
            if (byte == INIT_BYTE)
            {
                const unsigned char meta_flags_byte = udi_cdc_getc();
                const MetaFlags meta_flags = EXTRACT_META_FLAGS(meta_flags_byte);
                const unsigned char msg_size = meta_flags.MSG_SIZE-2;
                udi_cdc_read_buf(buffer, msg_size);
                HANDLE_MESSAGE(buffer, msg_size);
            }   
            CLEAR_BUFFER(buffer, USB_PACKET_SIZE_BYTES);
        }
    }

    void set_robot_actions()
    {
        set_robot_action(MOTORS, MOTOR_MOVE, &MOTOR_MOVE_F);
    }

    int main(void) 
    {
        init_robot_actions();
        set_robot_actions();
        usb_setup();
        enter_loop();
    }
    ```


## Maintainers

[@Conner Sommerfield](https://github.com/Repo-Factory/)  