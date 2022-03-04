# 2022-03-04 (Xen dt parsing)

Xen 에서 kernel device tree 를 파싱해서 사용할 때, reg property 값을 읽어오려면 아래의 코드를 참고할 수 있다.

// xen/arch/arm/smpboot.c

```c
        prop = dt_get_property(cpu, "reg", &reg_len);
        if ( !prop )
        {
            printk(XENLOG_WARNING "cpu node `%s`: has no reg property\n",
                   dt_node_full_name(cpu));
            continue;
        }

        if ( reg_len < dt_cells_to_size(dt_n_addr_cells(cpu)) )
        {
            printk(XENLOG_WARNING "cpu node `%s`: reg property too short\n",
                   dt_node_full_name(cpu));
            continue;
        }

        addr = dt_read_number(prop, dt_n_addr_cells(cpu));
```

이를 간단히 보면 `dt_get_property` 로 그 device tree 가 있는 메모리 주소 (DRAM) 를 알아내고, 그 주소의 값을 읽어오려면 `dt_read_number` 를 통해 접근해서 실제 값을 얻어온다.

```c
property_pointer = dt_get_property(dt_node, "reg", NULL);
property_value = dt_read_number(property_pointer, dt_n_addr_cells(dt_node));
```

이 외에도 xen 에서 dt parsing을 하기 위한 함수들은 아래와 같다.

// xen/include/xen/device_tree.h

```c
struct dt_device_node *dt_find_compatible_node(struct dt_device_node *from,
                                               const char *type,
                                               const char *compatible);

const void *dt_get_property(const struct dt_device_node *np,
                            const char *name, u32 *lenp);

const struct dt_property *dt_find_property(const struct dt_device_node *np,
                                           const char *name, u32 *lenp);

bool_t dt_property_read_u32(const struct dt_device_node *np,
                            const char *name, u32 *out_value);

bool_t dt_property_read_u64(const struct dt_device_node *np,
                            const char *name, u64 *out_value);

int dt_property_read_variable_u32_array(const struct dt_device_node *np,
                                        const char *propname, u32 *out_values,
                                        size_t sz_min, size_t sz_max);
```

