package com.kyc.service.impl;

import com.kyc.dto.KycResponseDto;
import com.kyc.model.KycApplication;
import com.kyc.model.KycStatus;
import com.kyc.repository.KycRepository;
import com.kyc.service.KycService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class KycServiceImpl implements KycService {

    @Autowired
    private KycRepository kycRepository;

    @Override
    public void approveKyc(Long kycId) {
        KycApplication application = kycRepository.findById(kycId)
                .orElseThrow(() -> new RuntimeException("KYC Application not found"));
        application.setStatus(KycStatus.APPROVED);
        kycRepository.save(application);
    }

    @Override
    public void rejectKyc(Long kycId) {
        KycApplication application = kycRepository.findById(kycId)
                .orElseThrow(() -> new RuntimeException("KYC Application not found"));
        application.setStatus(KycStatus.REJECTED);
        kycRepository.save(application);
    }

    @Override
    public List<KycResponseDto> getAllApplications() {
        return kycRepository.findAll().stream().map(application -> {
            KycResponseDto dto = new KycResponseDto();
            dto.setId(application.getId());
            dto.setStatus(application.getStatus());
            dto.setDocumentPath(application.getDocumentPath());
            dto.setSubmittedAt(application.getSubmittedAt());
            dto.setCustomerName(application.getCustomer().getName());
            return dto;
        }).collect(Collectors.toList());
    }
}
package com.kyc.service.impl;

import com.kyc.dto.CustomerDto;
import com.kyc.exception.ResourceNotFoundException;
import com.kyc.model.Customer;
import com.kyc.model.KycApplication;
import com.kyc.model.KycStatus;
import com.kyc.repository.CustomerRepository;
import com.kyc.repository.KycRepository;
import com.kyc.service.CustomerService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;

@Service
public class CustomerServiceImpl implements CustomerService {

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private KycRepository kycRepository;

    @Override
    public void submitKyc(CustomerDto customerDto) {
        Customer existingCustomer = customerRepository.findByEmail(customerDto.getEmail()).orElse(null);

        if (existingCustomer != null) {
            KycApplication existingApplication = existingCustomer.getKycApplication();
            if (existingApplication != null && existingApplication.getStatus() == KycStatus.APPROVED) {
                throw new RuntimeException("KYC already approved. Cannot modify.");
            }
        }

        Customer customer = new Customer();
        customer.setName(customerDto.getName());
        customer.setEmail(customerDto.getEmail());
        customer.setAddress(customerDto.getAddress());
        customer.setPhoneNumber(customerDto.getPhoneNumber());
        customer.setDateOfBirth(customerDto.getDateOfBirth());

        KycApplication kyc = new KycApplication();
        kyc.setStatus(KycStatus.PENDING);
        kyc.setDocumentPath(customerDto.getDocumentPath());
        kyc.setSubmittedAt(LocalDateTime.now());
        kyc.setCustomer(customer);

        customer.setKycApplication(kyc);

        customerRepository.save(customer);
    }

    @Override
    public KycStatus getApplicationStatus(String email) {
        Customer customer = customerRepository.findByEmail(email)
                .orElseThrow(() -> new ResourceNotFoundException("Customer not found with email: " + email));

        if (customer.getKycApplication() == null) {
            throw new ResourceNotFoundException("No KYC Application submitted.");
        }

        return customer.getKycApplication().getStatus();
    }
}
