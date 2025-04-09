package com.kyc.service.impl;

import com.kyc.dto.CustomerDto;
import com.kyc.dto.KycResponseDto;
import com.kyc.exception.ResourceNotFoundException;
import com.kyc.model.Customer;
import com.kyc.model.KycApplication;
import com.kyc.repository.CustomerRepository;
import com.kyc.repository.KycRepository;
import com.kyc.service.CustomerService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class CustomerServiceImpl implements CustomerService {

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private KycRepository kycRepository;

    @Override
    public void registerCustomer(CustomerDto dto) {
        Customer customer = new Customer();
        customer.setName(dto.getName());
        customer.setEmail(dto.getEmail());
        customer.setAddress(dto.getAddress());
        customer.setPhoneNumber(dto.getPhoneNumber());
        customer.setDateOfBirth(dto.getDateOfBirth());

        customerRepository.save(customer);
    }

    @Override
    public KycResponseDto getKycStatus(Long customerId) {
        KycApplication kyc = kycRepository.findByCustomerId(customerId)
                .orElseThrow(() -> new ResourceNotFoundException("KYC not found"));

        KycResponseDto dto = new KycResponseDto();
        dto.setId(kyc.getId());
        dto.setStatus(kyc.getStatus());
        dto.setDocumentPath(kyc.getDocumentPath());
        dto.setSubmittedAt(kyc.getSubmittedAt());
        return dto;
    }
}
