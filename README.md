@SpringBootTest
@AutoConfigureMockMvc
class EmployeeControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void shouldAddEmployee() throws Exception {
        Employee emp = new Employee(null, "John", "Doe", 9876543210L, 25L);
        mockMvc.perform(post("/api/v1/employee")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(emp)))
                .andExpect(status().isCreated());
    }

    @Test
    void shouldGetAllEmployees() throws Exception {
        mockMvc.perform(get("/api/v1/employee"))
                .andExpect(status().isOk());
    }

    @Test
    void shouldUpdateEmployee() throws Exception {
        // First add employee
        Employee emp = new Employee(null, "Jane", "Smith", 9876543211L, 30L);
        String json = objectMapper.writeValueAsString(emp);
        MvcResult result = mockMvc.perform(post("/api/v1/employee")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json)).andReturn();

        Employee created = objectMapper.readValue(result.getResponse().getContentAsString(), Employee.class);

        created.setFirstName("UpdatedName");
        mockMvc.perform(put("/api/v1/employee/" + created.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(created)))
                .andExpect(status().isOk());
    }

    @Test
    void shouldDeleteEmployee() throws Exception {
        Employee emp = new Employee(null, "Temp", "Del", 9876543212L, 28L);
        MvcResult result = mockMvc.perform(post("/api/v1/employee")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(emp))).andReturn();
        Employee created = objectMapper.readValue(result.getResponse().getContentAsString(), Employee.class);
        mockMvc.perform(delete("/api/v1/employee/" + created.getId()))
                .andExpect(status().isNoContent());
    }

    @Test
    void shouldReturnBadRequestForInvalidMobile() throws Exception {
        Employee emp = new Employee(null, "Err", "Bad", 123L, 20L);
        mockMvc.perform(post("/api/v1/employee")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(emp)))
                .andExpect(status().isBadRequest());
    }
}
