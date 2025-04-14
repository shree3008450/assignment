package com.example.kyc;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.mock.web.MockMultipartFile;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;

import static org.hamcrest.Matchers.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
public class BackendTest {

    @Autowired
    private MockMvc mockMvc;

    private Long registerCustomer() throws Exception {
        String customerJson = """
            {
                "name": "Test User",
                "email": "testuser@example.com",
                "address": "456 Lane",
                "phoneNumber": "9876543210",
                "dateOfBirth": "1990-01-01"
            }
        """;

        MvcResult result = mockMvc.perform(post("/api/customer/register")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(customerJson))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.customerId").exists())
                .andReturn();

        String json = result.getResponse().getContentAsString();
        JsonNode node = new ObjectMapper().readTree(json);
        return node.get("customerId").asLong();
    }

    @Test
    void testRegisterCustomer() throws Exception {
        registerCustomer(); // Validated in helper method
    }

    @Test
    void testSubmitKyc() throws Exception {
        Long customerId = registerCustomer();

        MockMultipartFile file = new MockMultipartFile(
                "document", "kyc.pdf", MediaType.APPLICATION_PDF_VALUE, "Dummy file".getBytes());

        mockMvc.perform(multipart("/api/kyc/submit/{customerId}", customerId)
                        .file(file)
                        .param("submittedDate", "2025-04-13"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.message").value("KYC submitted successfully."));
    }

    @Test
    void testGetKycStatus() throws Exception {
        Long customerId = registerCustomer();

        MockMultipartFile file = new MockMultipartFile(
                "document", "kyc.pdf", MediaType.APPLICATION_PDF_VALUE, "Dummy file".getBytes());

        mockMvc.perform(multipart("/api/kyc/submit/{customerId}", customerId)
                        .file(file)
                        .param("submittedDate", "2025-04-13"))
                .andExpect(status().isOk());

        mockMvc.perform(get("/api/customer/kyc-status/{id}", customerId))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.status").exists());
    }

    @Test
    void testApproveKyc() throws Exception {
        Long customerId = registerCustomer();

        MockMultipartFile file = new MockMultipartFile(
                "document", "kyc.pdf", MediaType.APPLICATION_PDF_VALUE, "Dummy file".getBytes());

        mockMvc.perform(multipart("/api/kyc/submit/{customerId}", customerId)
                        .file(file)
                        .param("submittedDate", "2025-04-13"))
                .andExpect(status().isOk());

        mockMvc.perform(put("/api/admin/approve/{id}", customerId)
                        .param("status", "APPROVED"))
                .andExpect(status().isOk())
                .andExpect(content().string("KYC status updated successfully"));
    }

    @Test
    void testRejectKyc() throws Exception {
        Long customerId = registerCustomer();

        MockMultipartFile file = new MockMultipartFile(
                "document", "kyc.pdf", MediaType.APPLICATION_PDF_VALUE, "Dummy file".getBytes());

        mockMvc.perform(multipart("/api/kyc/submit/{customerId}", customerId)
                        .file(file)
                        .param("submittedDate", "2025-04-13"))
                .andExpect(status().isOk());

        mockMvc.perform(put("/api/admin/reject/{id}", customerId)
                        .param("status", "REJECTED"))
                .andExpect(status().isOk())
                .andExpect(content().string("KYC status updated successfully"));
    }

    @Test
    void testGetAllApplications() throws Exception {
        Long customerId = registerCustomer();

        MockMultipartFile file = new MockMultipartFile(
                "document", "kyc.pdf", MediaType.APPLICATION_PDF_VALUE, "Dummy file".getBytes());

        mockMvc.perform(multipart("/api/kyc/submit/{customerId}", customerId)
                        .file(file)
                        .param("submittedDate", "2025-04-13"))
                .andExpect(status().isOk());

        mockMvc.perform(get("/api/admin/all-applications"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$").isArray())
                .andExpect(jsonPath("$[0].customer.id").value(customerId));
    }
}
