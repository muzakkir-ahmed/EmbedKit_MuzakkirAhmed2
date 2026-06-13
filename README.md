     #include <stdio.h>
     #include <stdint.h>
     #include <string.h>
 
    #define BUFFER_SIZE   8U
    #define BUFFER_MASK   (BUFFER_SIZE - 1U)
 
    #define RB_OK         0
    #define RB_ERR_FULL  -1
    #define RB_ERR_EMPTY -2
 
    typedef struct {
    uint8_t  data[BUFFER_SIZE];
    uint32_t head;
    uint32_t tail;
    uint32_t count;
    } RingBuffer;
 
    void rb_init(RingBuffer *rb)
    {
    memset(rb->data, 0x00, sizeof(rb->data));
    rb->head  = 0U;
    rb->tail  = 0U;
    rb->count = 0U;
    }
 
    uint8_t rb_is_full(const RingBuffer *rb)
    {
    return (rb->count == BUFFER_SIZE) ? 1U : 0U;
    }
 
    uint8_t rb_is_empty(const RingBuffer *rb)
    {
    return (rb->count == 0U) ? 1U : 0U;
    }
 
    uint32_t rb_count(const RingBuffer *rb)
    {
    return rb->count;
    }
 
    int rb_write(RingBuffer *rb, uint8_t byte)
    {
    if (rb_is_full(rb)) {
        return RB_ERR_FULL;
    }
    rb->data[rb->head] = byte;
    rb->head = (rb->head + 1U) & BUFFER_MASK;
    rb->count++;
    return RB_OK;
       }
 
     int rb_read(RingBuffer *rb, uint8_t *out)
      {
    if (rb_is_empty(rb)) {
        return RB_ERR_EMPTY;
     }
    *out = rb->data[rb->tail];
     rb->tail = (rb->tail + 1U) & BUFFER_MASK;
    rb->count--;
    return RB_OK;
    }
 
     static void demo_write(RingBuffer *rb, uint8_t byte)
     {
    int result = rb_write(rb, byte);
    if (result == RB_OK) {
        printf("[WRITE] 0x%02X -> OK (count=%u)%s\n",
               byte,
               rb_count(rb),
               rb_is_full(rb) ? " FULL" : "");
    } else {
        printf("[WRITE] 0x%02X -> FAIL (buffer full)\n", byte);
          }
      }
 
        static void demo_read(RingBuffer *rb)
    {
    uint8_t byte = 0x00;
    int result = rb_read(rb, &byte);
    if (result == RB_OK) {
        printf("[READ]  -> 0x%02X (count=%u)%s\n",
               byte,
               rb_count(rb),
               rb_is_empty(rb) ? " EMPTY" : "");
    } else {
        printf("[READ]  (empty) -> FAIL (buffer empty)\n");
      }
    }
 
     int main(void)
    {
    RingBuffer rb;
    rb_init(&rb);
    printf("========================================\n");
    printf("  EmbedKit Ring Buffer Demo\n");
    printf("  Author: Ahmed Muzakkir Uddin\n");
    printf("  Buffer capacity: %u bytes\n", BUFFER_SIZE);
    printf("========================================\n\n");
 
    printf("-- Step 1: Write 8 bytes (fill buffer) --\n");
    demo_write(&rb, 0x41);
    demo_write(&rb, 0x42);
    demo_write(&rb, 0x43);
    demo_write(&rb, 0x44);
    demo_write(&rb, 0x45);
    demo_write(&rb, 0x46);
    demo_write(&rb, 0x47);
    demo_write(&rb, 0x48);
    printf("Full=%u  Count=%u\n\n", rb_is_full(&rb), rb_count(&rb));
 
    printf("-- Step 2: Attempt overflow write --\n");
    demo_write(&rb, 0x99);
    printf("\n");
 
    printf("-- Step 3: Read 3 bytes --\n");
    demo_read(&rb);
    demo_read(&rb);
    demo_read(&rb);
    printf("Count after 3 reads=%u\n\n", rb_count(&rb));
 
    printf("-- Step 4: Write 3 new bytes (wrap-around) --\n");
    demo_write(&rb, 0x49);
    demo_write(&rb, 0x4A);
    demo_write(&rb, 0x4B);
    printf("Count=%u  Full=%u\n\n", rb_count(&rb), rb_is_full(&rb));
 
    printf("-- Step 5: Read all 8 bytes --\n");
    demo_read(&rb);
    demo_read(&rb);
    demo_read(&rb);
    demo_read(&rb);
    demo_read(&rb);
    demo_read(&rb);
    demo_read(&rb);
    demo_read(&rb);
    printf("Empty=%u  Count=%u\n\n", rb_is_empty(&rb), rb_count(&rb));
 
    printf("-- Step 6: Attempt underflow read --\n");
    demo_read(&rb);
 
    printf("\n========================================\n");
    printf("  All steps complete.\n");
    printf("========================================\n");
 
    return 0;
                     }
   
  
