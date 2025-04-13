package org.example;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import org.example.dto.CustomerDto;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.mock.web.MockMultipartFile;
import org.springframework.test.web.servlet.MockMvc;

import java.time.LocalDate;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest(classes = Main.class)
@AutoConfigureMockMvc
public class BackendTest {

    @Autowired
    private MockMvc mockMvc;

    private ObjectMapper objectMapper;

    @BeforeEach
    void setup() {
        objectMapper = new ObjectMapper();
        objectMapper.registerModule(new JavaTimeModule());
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    }

    @Test
    void testRegisterCustomer() throws Exception {
        CustomerDto customerDto = new CustomerDto();
        customerDto.setName("John");
        customerDto.setEmail("john@mail.com");
        customerDto.setAddress("123 Street");
        customerDto.setPhoneNumber("1234567890");
        customerDto.setDateOfBirth(LocalDate.of(1990, 1, 1));

        mockMvc.perform(post("/api/customers/register")
                .content(objectMapper.writeValueAsString(customerDto))
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").exists());
    }

    @Test
    void testSubmitKyc() throws Exception {
        MockMultipartFile file = new MockMultipartFile("document", "doc.pdf", MediaType.APPLICATION_PDF_VALUE, "Dummy PDF Content".getBytes());

        mockMvc.perform(multipart("/api/kyc/submit/1").file(file))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.message").value("KYC submitted successfully"));
    }

    @Test
    void testGetKycStatus() throws Exception {
        mockMvc.perform(get("/api/kyc/1/status"))
                .andExpect(status().isOk());
    }

    @Test
    void testApproveKyc() throws Exception {
        mockMvc.perform(put("/api/admin/approve/1")
                .param("status", "APPROVED"))
                .andExpect(status().isOk())
                .andExpect(content().string("KYC status updated successfully"));
    }

    @Test
    void testRejectKyc() throws Exception {
        mockMvc.perform(put("/api/admin/reject/1")
                .param("status", "REJECTED"))
                .andExpect(status().isOk())
                .andExpect(content().string("KYC status updated successfully"));
    }
}
