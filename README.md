package com.kyc.model;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class KycApplication {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String fullName;
    private String dateOfBirth;
    private String address;

    private String documentPath;

    @Enumerated(EnumType.STRING)
    private KycStatus status;

    private String rejectionReason;

    @OneToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
}




package com.kyc.dto;

import lombok.Data;

@Data
public class CustomerDto {
    private String username;
    private String email;
}



package com.kyc.dto;

import lombok.Data;
import org.springframework.web.multipart.MultipartFile;

@Data
public class KycRequestDto {
    private String username;
    private String fullName;
    private String dateOfBirth;
    private String address;
    private MultipartFile document;
}




package com.kyc.dto;

import lombok.Builder;
import lombok.Data;

@Data
@Builder
public class KycResponseDto {
    private Long applicationId;
    private String fullName;
    private String status;
    private String rejectionReason;
}



package com.kyc.repository;

import com.kyc.model.Customer;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface CustomerRepository extends JpaRepository<Customer, Long> {
    Optional<Customer> findByUsername(String username);
}



package com.kyc.repository;

import com.kyc.model.KycApplication;
import org.springframework.data.jpa.repository.JpaRepository;

public interface KycRepository extends JpaRepository<KycApplication, Long> {
}



package com.kyc.service;

import com.kyc.dto.CustomerDto;
import com.kyc.model.Customer;

public interface CustomerService {
    Customer registerCustomer(CustomerDto dto);
    Customer getByUsername(String username);
}


package com.kyc.service.impl;

import com.kyc.dto.CustomerDto;
import com.kyc.model.Customer;
import com.kyc.repository.CustomerRepository;
import com.kyc.service.CustomerService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class CustomerServiceImpl implements CustomerService {

    private final CustomerRepository customerRepository;

    @Override
    public Customer registerCustomer(CustomerDto dto) {
        Customer customer = Customer.builder()
                .username(dto.getUsername())
                .email(dto.getEmail())
                .build();
        return customerRepository.save(customer);
    }

    @Override
    public Customer getByUsername(String username) {
        return customerRepository.findByUsername(username)
                .orElseThrow(() -> new RuntimeException("Customer not found"));
    }
}





package com.kyc.service;

import com.kyc.dto.KycRequestDto;
import com.kyc.dto.KycResponseDto;

import java.util.List;

public interface KycService {
    KycResponseDto submitKyc(KycRequestDto dto);
    List<KycResponseDto> getAllApplications();
    void approveApplication(Long id);
    void rejectApplication(Long id, String reason);
}



package com.kyc.service.impl;

import com.kyc.dto.KycRequestDto;
import com.kyc.dto.KycResponseDto;
import com.kyc.model.*;
import com.kyc.repository.*;
import com.kyc.service.*;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.List;

@Service
@RequiredArgsConstructor
public class KycServiceImpl implements KycService {

    private final KycRepository kycRepository;
    private final CustomerService customerService;

    @Override
    public KycResponseDto submitKyc(KycRequestDto dto) {
        Customer customer = customerService.getByUsername(dto.getUsername());

        MultipartFile doc = dto.getDocument();
        String pathStr = "uploads/" + System.currentTimeMillis() + "_" + doc.getOriginalFilename();

        try {
            Path path = Paths.get(pathStr);
            Files.createDirectories(path.getParent());
            doc.transferTo(path);
        } catch (Exception e) {
            throw new RuntimeException("Error saving document", e);
        }

        KycApplication kyc = KycApplication.builder()
                .fullName(dto.getFullName())
                .dateOfBirth(dto.getDateOfBirth())
                .address(dto.getAddress())
                .documentPath(pathStr)
                .status(KycStatus.PENDING)
                .customer(customer)
                .build();

        kycRepository.save(kyc);
        customer.setKycApplication(kyc);

        return KycResponseDto.builder()
                .applicationId(kyc.getId())
                .fullName(kyc.getFullName())
                .status(kyc.getStatus().name())
                .build();
    }

    @Override
    public List<KycResponseDto> getAllApplications() {
        return kycRepository.findAll().stream().map(app -> KycResponseDto.builder()
                .applicationId(app.getId())
                .fullName(app.getFullName())
                .status(app.getStatus().name())
                .rejectionReason(app.getRejectionReason())
                .build()).toList();
    }

    @Override
    public void approveApplication(Long id) {
        KycApplication app = kycRepository.findById(id).orElseThrow();
        app.setStatus(KycStatus.APPROVED);
        kycRepository.save(app);
    }

    @Override
    public void rejectApplication(Long id, String reason) {
        KycApplication app = kycRepository.findById(id).orElseThrow();
        app.setStatus(KycStatus.REJECTED);
        app.setRejectionReason(reason);
        kycRepository.save(app);
    }
}




package com.kyc.controller;

import com.kyc.dto.CustomerDto;
import com.kyc.model.Customer;
import com.kyc.service.CustomerService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/customers")
@RequiredArgsConstructor
public class CustomerController {

    private final CustomerService customerService;

    @PostMapping("/register")
    public Customer register(@RequestBody CustomerDto dto) {
        return customerService.registerCustomer(dto);
    }

    @GetMapping("/{username}")
    public Customer getCustomer(@PathVariable String username) {
        return customerService.getByUsername(username);
    }
}




package com.kyc.controller;

import com.kyc.dto.KycResponseDto;
import com.kyc.service.KycService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/admin")
@RequiredArgsConstructor
public class AdminController {

    private final KycService kycService;

    @GetMapping("/kyc-applications")
    public List<KycResponseDto> getAllApplications() {
        return kycService.getAllApplications();
    }

    @PutMapping("/approve/{id}")
    public void approve(@PathVariable Long id) {
        kycService.approveApplication(id);
    }

    @PutMapping("/reject/{id}")
    public void reject(@PathVariable Long id, @RequestParam String reason) {
        kycService.rejectApplication(id, reason);
    }
}
