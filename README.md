package com.kyc.service.impl;

import com.kyc.dto.KycRequestDto;
import com.kyc.dto.KycResponseDto;
import com.kyc.exception.ResourceNotFoundException;
import com.kyc.model.Customer;
import com.kyc.model.KycApplication;
import com.kyc.model.KycStatus;
import com.kyc.repository.CustomerRepository;
import com.kyc.repository.KycRepository;
import com.kyc.service.KycService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.List;
import java.util.stream.Collectors;

@Service
public class KycServiceImpl implements KycService {

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private KycRepository kycRepository;

    @Override
    public void submitKyc(KycRequestDto dto) {
        Customer customer = customerRepository.findById(dto.getCustomerId())
                .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));

        KycApplication existing = kycRepository.findByCustomerId(customer.getId()).orElse(null);

        if (existing != null && existing.getStatus() != KycStatus.REJECTED) {
            throw new IllegalStateException("You can't modify KYC unless it's rejected");
        }

        KycApplication kyc = new KycApplication();
        kyc.setCustomer(customer);
        kyc.setDocumentPath(dto.getDocumentPath());
        kyc.setStatus(KycStatus.PENDING);
        kyc.setSubmittedAt(LocalDateTime.now());

        kycRepository.save(kyc);
    }

    @Override
    public void approveKyc(Long kycId) {
        KycApplication kyc = kycRepository.findById(kycId)
                .orElseThrow(() -> new ResourceNotFoundException("KYC not found"));
        kyc.setStatus(KycStatus.APPROVED);
        kycRepository.save(kyc);
    }

    @Override
    public void rejectKyc(Long kycId) {
        KycApplication kyc = kycRepository.findById(kycId)
                .orElseThrow(() -> new ResourceNotFoundException("KYC not found"));
        kyc.setStatus(KycStatus.REJECTED);
        kycRepository.save(kyc);
    }

    @Override
    public List<KycResponseDto> getAllApplications() {
        List<KycApplication> list = kycRepository.findAll();
        return list.stream().map(kyc -> {
            KycResponseDto dto = new KycResponseDto();
            dto.setId(kyc.getId());
            dto.setCustomerId(kyc.getCustomer().getId());
            dto.setCustomerName(kyc.getCustomer().getName());
            dto.setStatus(kyc.getStatus());
            dto.setDocumentPath(kyc.getDocumentPath());
            dto.setSubmittedAt(kyc.getSubmittedAt());
            return dto;
        }).collect(Collectors.toList());
    }
}











package com.kyc.controller;

import com.kyc.dto.KycResponseDto;
import com.kyc.service.KycService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/admin")
public class AdminController {

    @Autowired
    private KycService kycService;

    @PostMapping("/approve/{kycId}")
    public String approveKyc(@PathVariable Long kycId) {
        kycService.approveKyc(kycId);
        return "KYC Approved";
    }

    @PostMapping("/reject/{kycId}")
    public String rejectKyc(@PathVariable Long kycId) {
        kycService.rejectKyc(kycId);
        return "KYC Rejected";
    }

    @GetMapping("/all-applications")
    public List<KycResponseDto> getAllApplications() {
        return kycService.getAllApplications();
    }
}
