tent, style: TextStyle(color: Colors.grey.shade400, fontSize: 13, height: 1.6)),
        ],
      ),
    );
  }

  Widget _statCard(String value, String label, IconData icon, Color color) {
    return Container(
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: const Color(0xFF141414),
        borderRadius: BorderRadius.circular(14),
        border: Border.all(color: Colors.white10),
      ),
      child: Column(
        children: [
          Icon(icon, color: color, size: 22),
          const SizedBox(height: 6),
          Text(value, style: TextStyle(color: color, fontWeight: FontWeight.bold, fontSize: 16)),
          const SizedBox(height: 2),
          Text(label, style: TextStyle(color: Colors.grey.shade500, fontSize: 10), textAlign: TextAlign.center),
        ],
      ),
    );
  }

  Widget _sectionHeader(IconData icon, String title, Color color) {
    return Row(
      children: [
        Icon(icon, color: color, size: 20),
        const SizedBox(width: 8),
        Text(title, style: TextStyle(color: color, fontSize: 16, fontWeight: FontWeight.bold)),
      ],
    );
  }

  Widget _legalTile(IconData icon, String text) {
    return Container(
      margin: const EdgeInsets.only(bottom: 8),
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: const Color(0xFF141414),
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: Colors.white10),
      ),
      child: Row(
        children: [
          Icon(icon, color: Colors.grey, size: 18),
          const SizedBox(width: 12),
          Expanded(child: Text(text, style: TextStyle(color: Colors.grey.shade400, fontSize: 13))),
        ],
      ),
    );
  }
}

و

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../cubit/driver_cubit.dart';
import '../cubit/driver_state.dart';
import '../widgets/nearby_request_card.dart';
import '../widgets/driver_status_badge.dart';
import '../widgets/notification_banner.dart';
import 'trip_view_screen.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'personal_info_screen.dart';
import 'history_screen.dart';
import 'wallet_screen.dart';
import 'settings_screen.dart';
import '../../../core/localization/app_localizations.dart';
import '../../../core/localization/localization_cubit.dart';
import '../../../core/theme/app_colors.dart';

class DriverHomeScreen extends StatelessWidget {
  final String driverId;

  const DriverHomeScreen({super.key, required this.driverId});

  @override
  Widget build(BuildContext context) {
    final lang = context.watch<LocalizationCubit>().state.languageCode;
    return Scaffold(
      backgroundColor: AppColors.scaffold(context),
      endDrawer: _buildDrawer(context, lang),
      body: SafeArea(
        child: BlocConsumer<DriverCubit, DriverState>(
          listener: (context, state) {
            if (state is DriverLoaded) {
              // Show error snackbar
              if (state.errorMessage != null) {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text(state.errorMessage!),
                    backgroundColor: Colors.redAccent,
                    behavior: SnackBarBehavior.floating,
                  ),
                );
                context.read<DriverCubit>().clearError();
              }

              // Show notifications as banners
              if (state.notifications.isNotEmpty &&
                  !state.notifications.first.isRead) {
                NotificationBanner.show(context, state.notifications.first);
              }

              // Navigate to trip view when active trip exists AND we are currently on the home screen
              final isCurrentRoute = ModalRoute.of(context)?.isCurrent ?? false;
              if (isCurrentRoute && 
                  state.activeTrip != null &&
                  (state.profile.status == 'busy' ||
                      state.profile.status == 'on_trip')) {
                Navigator.of(context).push(
                  MaterialPageRoute(
                    builder: (_) => BlocProvider.value(
                      value: context.read<DriverCubit>(),
                      child: TripViewScreen(
                        tripId: state.activeTrip!.id,
                        driverId: driverId,
                      ),
                    ),
                  ),
                );
              }
            }
          },
          builder: (context, state) {
            if (state is DriverInitial || state is DriverLoading) {
              return const Center(
                child: CircularProgressIndicator(color: Color(0xFFF5D142)),
              );
            } else if (state is DriverError) {
              return Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    const Icon(Icons.error_outline,
                        size: 48, color: Colors.redAccent),
                    const SizedBox(height: 16),
                    Text('${AppLocalizations.translate('error', lang)}${state.message}',
                        style: const TextStyle(color: Colors.red)),
                    const SizedBox(height: 16),
                    ElevatedButton(
                      style: ElevatedButton.styleFrom(
                        backgroundColor: const Color(0xFFF5D142),
                        foregroundColor: Colors.black,
                      ),
                      onPressed: () {
                        context.read<DriverCubit>().loadDriver(driverId);
                      },
                      child: Text(AppLocalizations.translate('retry', lang)),
                    ),
                  ],
                ),
              );
            } else if (state is DriverLoaded) {
              return _buildLoadedView(context, state, lang);
            }
            return const SizedBox.shrink();
          },
        ),
      ),
    );
  }

  Widget _buildLoadedView(BuildContext context, DriverLoaded state, String lang) {
    final isOnline = state.profile.status == 'online' ||
        state.profile.status == 'available';
    final isBlocked = state.profile.isBlocked;

    return Column(
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: [
        _buildCustomHeader(context, isOnline, state, lang),
        // Blocked banner
        if (isBlocked) _buildBlockedBanner(lang),
        // Low balance warning
        if (state.profile.isLowBalance && !isBlocked)
          _buildLowBalanceBanner(state.profile.walletBalance, lang),
        const SizedBox(height: 16),
        // Wallet summary
        _buildWalletSummary(context, state, lang),
        const SizedBox(height: 20),
        Padding(
          padding: const EdgeInsets.symmetric(horizontal: 20),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text(
                AppLocalizations.translate('requests_list', lang),
                style: const TextStyle(
                  fontSize: 22,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFFF5D142),
                ),
              ),
              DriverStatusBadge(status: state.profile.status),
            ],
          ),
        ),
        const SizedBox(height: 12),
        Expanded(
          child: isBlocked
              ? _buildBlockedView(lang)
              : isOnline
                  ? _buildOnlineView(context, state, lang)
                  : _buildOfflineView(lang),
        ),
      ],
    );
  }

  Widget _buildCustomHeader(
      BuildContext context, bool isOnline, DriverLoaded state, String lang) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 10),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          // Custom Toggle
          GestureDetector(
            onTap: () {
              context.read<DriverCubit>().toggleStatus(driverId, !isOnline);
            },
            child: AnimatedContainer(
              duration: const Duration(milliseconds: 300),
              padding:
                  const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
              decoration: BoxDecoration(
                color: AppColors.surface(context),
                borderRadius: BorderRadius.circular(12),
                border: Border.all(
                  color: isOnline
                      ? const Color(0xFFF5D142).withValues(alpha: 0.4)
                      : Colors.white12,
                  width: 1,
                ),
              ),
              child: Row(
                children: [
                  Text(
                    '- ${AppLocalizations.translate(isOnline ? 'online' : 'offline', lang).toUpperCase()}',
                    style: TextStyle(
                      color:
                          isOnline ? const Color(0xFFF5D142) : Colors.grey,
                      fontWeight: FontWeight.bold,
                      fontSize: 18,
                      letterSpacing: 0.5,
                    ),
                  ),
                  const SizedBox(width: 14),
                  Container(
                    width: 14,
                    height: 14,
                    decoration: BoxDecoration(
                      shape: BoxShape.circle,
                      color: isOnline
                          ? Colors.greenAccent.shade400
                          : Colors.grey.shade700,
                      boxShadow: isOnline
                          ? [
                              BoxShadow(
                                color: Colors.greenAccent.withValues(alpha: 0.5),
                                blurRadius: 8,
                                spreadRadius: 2,
                              )
                            ]
                          : [],
                    ),
                  ),
                ],
              ),
            ),
          ),
          // Menu + notification badge
          Builder(builder: (ctx) {
            return Stack(
              children: [
                IconButton(
                  icon: const Icon(Icons.menu,
                      color: Color(0xFFF5D142), size: 32),
                  onPressed: () {
                    Scaffold.of(ctx).openEndDrawer();
                  },
                ),
                if (state.unreadNotificationCount > 0)
                  Positioned(
                    right: 6,
                    top: 6,
                    child: Container(
                      padding: const EdgeInsets.all(4),
                      decoration: const BoxDecoration(
                        color: Colors.redAccent,
                        shape: BoxShape.circle,
                      ),
                      child: Text(
                        '${state.unreadNotificationCount}',
                        style: const TextStyle(
                            color: Colors.white, fontSize: 10),
                      ),
                    ),
                  ),
              ],
            );
          }),
        ],
      ),
    );
  }

  Widget _buildWalletSummary(BuildContext context, DriverLoaded state, String lang) {
    const goldColor = Color(0xFFF5D142);
    return GestureDetector(
      onTap: () {
        Navigator.of(context).push(
          MaterialPageRoute(
            builder: (_) => BlocProvider.value(
              value: context.read<DriverCubit>(),
              child: WalletScreen(driverId: driverId),
            ),
          ),
        );
      },
      child: Container(
        margin: const EdgeInsets.symmetric(horizontal: 20),
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: AppColors.surface(context),
          borderRadius: BorderRadius.circular(14),
          border: Border.all(color: goldColor.withValues(alpha: 0.3)),
        ),
        child: Row(
          children: [
            const Icon(Icons.account_balance_wallet,
                color: goldColor, size: 28),
            const SizedBox(width: 14),
            Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(AppLocalizations.translate('wallet', lang),
                    style: TextStyle(
                        color: Colors.grey.shade400, fontSize: 12)),
                Text(
                  '${state.profile.walletBalance.toStringAsFixed(2)} MRU',
                  style: const TextStyle(
                    color: goldColor,
                    fontSize: 22,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
            const Spacer(),
            Text('${state.profile.totalTrips} ${AppLocalizations.translate('trips', lang)}',
                style:
                    TextStyle(color: Colors.grey.shade500, fontSize: 13)),
            const SizedBox(width: 8),
            Icon(Icons.chevron_right, color: Colors.grey.shade600),
          ],
        ),
      ),
    );
  }

  Widget _buildBlockedBanner(String lang) {
    return Container(
      margin: const EdgeInsets.symmetric(horizontal: 20, vertical: 8),
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: Colors.red.shade900.withValues(alpha: 0.3),
        borderRadius: BorderRadius.circular(10),
        border: Border.all(color: Colors.redAccent.withValues(alpha: 0.5)),
      ),
      child: Row(
        children: [
          const Icon(Icons.block, color: Colors.redAccent, size: 24),
          const SizedBox(width: 12),
          Expanded(
            child: Text(
              AppLocalizations.translate('account_blocked_msg', lang),
              style: const TextStyle(color: Colors.redAccent, fontSize: 13),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildLowBalanceBanner(double balance, String lang) {
    return Container(
      margin: const EdgeInsets.symmetric(horizontal: 20, vertical: 8),
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: Colors.orange.shade900.withValues(alpha: 0.2),
        borderRadius: BorderRadius.circular(10),
        border: Border.all(color: Colors.orangeAccent.withValues(alpha: 0.4)),
      ),
      child: Row(
        children: [
          const Icon(Icons.warning_amber, color: Colors.orangeAccent, size: 24),
          const SizedBox(width: 12),
          Expanded(
            child: Text(
              '${AppLocalizations.translate('low_balance_msg', lang)} ($balance MRU)',
              style: const TextStyle(color: Colors.orangeAccent, fontSize: 13),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildDrawer(BuildContext context, String lang) {
    final driverId = FirebaseAuth.instance.currentUser?.uid ?? '';
    return Drawer(
      backgroundColor: AppColors.scaffold(context),
      child: SafeArea(
        child: ListView(
          padding: const EdgeInsets.all(20),
          children: [
            const SizedBox(height: 20),
            _drawerItem(
                context: context,
                icon: Icons.history,
                text: AppLocalizations.translate('history', lang),
                onTap: () {
                  Navigator.pop(context);
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (_) => BlocProvider.value(
                        value: context.read<DriverCubit>(),
                        child: HistoryScreen(driverId: driverId),
                      ),
                    ),
                  );
                }),
            _drawerItem(
                context: context,
                icon: Icons.account_balance_wallet_outlined,
                text: AppLocalizations.translate('my_wallet', lang),
                onTap: () {
                  Navigator.pop(context);
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (_) => BlocProvider.value(
                        value: context.read<DriverCubit>(),
                        child: WalletScreen(driverId: driverId),
                      ),
                    ),
                  );
                }),
            _drawerItem(
                context: context,
                icon: Icons.settings_outlined,
                text: AppLocalizations.translate('settings', lang),
                onTap: () {
                  Navigator.pop(context);
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (_) => BlocProvider.value(
                        value: context.read<DriverCubit>(),
                        child: SettingsScreen(driverId: driverId),
                      ),
                    ),
                  );
                }),
          ],
        ),
      ),
    );
  }

  Widget _drawerItem(
      {required BuildContext context,
      required IconData icon,
      required String text,
      required VoidCallback onTap}) {
    return Container(
      margin: const EdgeInsets.only(bottom: 16),
      decoration: BoxDecoration(
        color: AppColors.surface(context),
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: const Color(0xFFF5D142), width: 1),
        boxShadow: [
          BoxShadow(
            color: const Color(0xFFF5D142).withValues(alpha: 0.1),
            blurRadius: 5,
            spreadRadius: 1,
          )
        ],
      ),
      child: ListTile(
        leading: Icon(icon, color: const Color(0xFFF5D142)),
        title: Text(
          text,
          style: const TextStyle(color: Colors.white, fontSize: 18),
        ),
        onTap: onTap,
      ),
    );
  }

  Widget _buildOfflineView(String lang) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.power_settings_new, size: 64, color: Colors.grey),
          const SizedBox(height: 16),
          Text(
            AppLocalizations.translate('you_are_offline', lang),
            style: TextStyle(fontSize: 18, color: Colors.grey.shade400),
          ),
          const SizedBox(height: 8),
          Text(
            AppLocalizations.translate('activate_to_receive', lang),
            style: TextStyle(fontSize: 13, color: Colors.grey.shade600),
          ),
        ],
      ),
    );
  }

  Widget _buildBlockedView(String lang) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.block, size: 64, color: Colors.redAccent),
          const SizedBox(height: 16),
          Text(
            AppLocalizations.translate('access_suspended', lang),
            style: TextStyle(fontSize: 18, color: Colors.grey.shade400),
          ),
        ],
      ),
    );
  }

  Widget _buildOnlineView(BuildContext context, DriverLoaded state, String lang) {
    if (state.nearbyRequests.isEmpty) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const CircularProgressIndicator(color: Color(0xFFF5D142)),
            const SizedBox(height: 16),
            Text(AppLocalizations.translate('waiting_for_requests', lang),
                style: TextStyle(color: Colors.grey.shade400)),
          ],
        ),
      );
    }
    return ListView.builder(
      padding: const EdgeInsets.only(bottom: 20),
      itemCount: state.nearbyRequests.length,
      itemBuilder: (context, index) {
        final request = state.nearbyRequests[index];
        return NearbyRequestCard(
          request: request,
          onAccept: () {
            context.read<DriverCubit>().acceptTrip(driverId, request.id);
          },
          onDismiss: () {
            context.read<DriverCubit>().dismissTrip(request.id);
          },
        );
      },
    );
  }
}


و


import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../core/localization/app_localizations.dart';
import '../../../core/localization/localization_cubit.dart';
import 'package:url_launcher/url_launcher.dart';
import '../../../core/theme/app_colors.dart';

class HelpSupportScreen extends StatelessWidget {
  const HelpSupportScreen({super.key});

  Future<void> _launchUrl(String url, BuildContext context) async {
    final uri = Uri.parse(url);
    if (await canLaunchUrl(uri)) {
      await launchUrl(uri, mode: LaunchMode.externalApplication);
    }
  }

  Future<void> _sendEmail(String email, BuildContext context) async {
    final uri = Uri(scheme: 'mailto', path: email, query: 'subject=Support Tawseel');
    if (await canLaunchUrl(uri)) {
      await launchUrl(uri);
    }
  }

  Future<void> _callPhone(String phone, BuildContext context) async {
    final uri = Uri.parse('tel:$phone');
    if (await canLaunchUrl(uri)) {
      await launchUrl(uri);
    }
  }

  @override
  Widget build(BuildContext context) {
    final lang = context.watch<LocalizationCubit>().state.languageCode;
    const goldColor = Color(0xFFF5D142);

    final faqs = [
      {
        'q': AppLocalizations.translate('faq_q1', lang),
        'a': AppLocalizations.translate('faq_a1', lang),
      },
      {
        'q': AppLocalizations.translate('faq_q2', lang),
        'a': AppLocalizations.translate('faq_a2', lang),
      },
      {
        'q': AppLocalizations.translate('faq_q3', lang),
        'a': AppLocalizations.translate('faq_a3', lang),
      },
      {
        'q': AppLocalizations.translate('faq_q4', lang),
        'a': AppLocalizations.translate('faq_a4', lang),
      },
    ];

    return Scaffold(
      backgroundColor: AppColors.scaffold(context),
      appBar: AppBar(
        backgroundColor: AppColors.scaffold(context),
        foregroundColor: Colors.white,
        elevation: 0,
        title: Text(AppLocalizations.translate('help_support', lang),
            style: const TextStyle(fontWeight: FontWeight.bold)),
        centerTitle: true,
        leading: IconButton(
          icon: const Icon(Icons.arrow_back_ios),
          onPressed: () => Navigator.pop(context),
        ),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Contact Us
            _sectionHeader(Icons.headset_mic_outlined, AppLocalizations.translate('contact_us', lang), goldColor),
            const SizedBox(height: 12),
            Row(
              children: [
                Expanded(
                  child: _contactCard(
                    icon: Icons.phone,
                    label: AppLocalizations.translate('call_us', lang),
                    sublabel: '+222 XX XX XX XX',
                    color: Colors.greenAccent,
                    onTap: () => _callPhone('+222XXXXXXXX', context),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: _contactCard(
                    icon: Icons.email_outlined,
                    label: AppLocalizations.translate('email_us', lang),
                    sublabel: 'support@tawseel.app',
                    color: Colors.blueAccent,
                    onTap: () => _sendEmail('support@tawseel.app', context),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 28),
            // FAQ Section
            _sectionHeader(Icons.quiz_outlined, AppLocalizations.translate('faq', lang), goldColor),
            const SizedBox(height: 12),
            ...faqs.map((faq) => _faqTile(faq['q']!, faq['a']!)),
          ],
        ),
      ),
    );
  }

  Widget _sectionHeader(IconData icon, String title, Color color) {
    return Row(
      children: [
        Icon(icon, color: color, size: 22),
        const SizedBox(width: 10),
        Text(title, style: TextStyle(color: color, fontSize: 17, fontWeight: FontWeight.bold)),
      ],
    );
  }

  Widget _contactCard({
    required IconData icon,
    required String label,
    required String sublabel,
    required Color color,
    required VoidCallback onTap,
  }) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: const Color(0xFF141414),
          borderRadius: BorderRadius.circular(14),
          border: Border.all(color: color.withValues(alpha: 0.3)),
        ),
        child: Column(
          children: [
            Icon(icon, color: color, size: 28),
            const SizedBox(height: 8),
            Text(label, style: const TextStyle(color: Colors.white, fontWeight: FontWeight.bold, fontSize: 13)),
            const SizedBox(height: 4),
            Text(sublabel, style: TextStyle(color: Colors.grey.shade500, fontSize: 11)),
          ],
        ),
      ),
    );
  }

  Widget _faqTile(String question, String answer) {
    return Container(
      margin: const EdgeInsets.only(bottom: 8),
      decoration: BoxDecoration(
        color: const Color(0xFF141414),
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: Colors.white10),
      ),
      child: Theme(
        data: ThemeData(dividerColor: Colors.transparent),
        child: ExpansionTile(
          iconColor: const Color(0xFFF5D142),
          collapsedIconColor: Colors.grey,
          title: Text(question, style: const TextStyle(color: Colors.white, fontSize: 14, fontWeight: FontWeight.w500)),
          children: [
            Padding(
              padding: const EdgeInsets.fromLTRB(16, 0, 16, 16),
              child: Text(answer, style: TextStyle(color: Colors.grey.shade400, fontSize: 13, height: 1.5)),
            ),
          ],
        ),
      ),
    );
  }
}


و

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../cubit/driver_cubit.dart';
import '../cubit/driver_state.dart';
import '../widgets/trip_history_card.dart';
import '../../../core/localization/app_localizations.dart';
import '../../../core/localization/localization_cubit.dart';
import '../../../core/theme/app_colors.dart';

class HistoryScreen extends StatelessWidget {
  final String driverId;

  const HistoryScreen({super.key, required this.driverId});

  @override
  Widget build(BuildContext context) {
    final lang = context.watch<LocalizationCubit>().state.languageCode;
    const goldColor = Color(0xFFF5D142);

    return Scaffold(
      backgroundColor: AppColors.scaffold(context),
      appBar: AppBar(
        backgroundColor: AppColors.scaffold(context),
        foregroundColor: Colors.white,
        elevation: 0,
        title: Text(AppLocalizations.translate('history', lang),
            style: const TextStyle(fontWeight: FontWeight.bold)),
        centerTitle: true,
        leading: IconButton(
          icon: const Icon(Icons.arrow_back_ios),
          onPressed: () => Navigator.pop(context),
        ),
      ),
      body: BlocBuilder<DriverCubit, DriverState>(
        builder: (context, state) {
          if (state is! DriverLoaded) {
            return const Center(
              child: CircularProgressIndicator(color: goldColor),
            );
          }

          final history = state.tripHistory;

          // Stats
          final totalTrips = history.length;
          double totalEarnings = 0;
          double totalCommission = 0;
          for (final trip in history) {
            totalEarnings += (trip.fare ?? trip.estimatedPrice);
            totalCommission += (trip.commission ?? 0);
          }
          final netEarnings = totalEarnings - totalCommission;

          return Column(
            children: [
              // Stats summary
              Container(
                margin: const EdgeInsets.all(20),
                padding: const EdgeInsets.all(20),
                decoration: BoxDecoration(
                  color: AppColors.surface(context),
                  borderRadius: BorderRadius.circular(16),
                  border: Border.all(color: goldColor.withValues(alpha: 0.3)),
                ),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  children: [
                    _statItem(AppLocalizations.translate('trips', lang), '$totalTrips', goldColor),
                    Container(width: 1, height: 40, color: Colors.white12),
                    _statItem(AppLocalizations.translate('gross_income', lang),
                        '${totalEarnings.toStringAsFixed(2)} MRU', Colors.white),
                    Container(width: 1, height: 40, color: Colors.white12),
                    _statItem(AppLocalizations.translate('net_income', lang),
                        '${netEarnings.toStringAsFixed(2)} MRU', goldColor),
                  ],
                ),
              ),
              // Trip list
              Expanded(
                child: history.isEmpty
                    ? Center(
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.history,
                                size: 64, color: Colors.grey.shade700),
                            const SizedBox(height: 16),
                            Text(AppLocalizations.translate('no_completed_trips', lang),
                                style: TextStyle(
                                    color: Colors.grey.shade500,
                                    fontSize: 16)),
                          ],
                        ),
                      )
                    : ListView.builder(
                        padding: const EdgeInsets.only(bottom: 20),
                        itemCount: history.length,
                        itemBuilder: (context, index) {
                          return TripHistoryCard(trip: history[index]);
                        },
                      ),
              ),
            ],
          );
        },
      ),
    );
  }

  Widget _statItem(String label, String value, Color valueColor) {
    return Column(
      children: [
        Text(label,
            style: TextStyle(color: Colors.grey.shade500, fontSize: 12)),
        const SizedBox(height: 6),
        Text(value,
            style: TextStyle(
                color: valueColor,
                fontWeight: FontWeight.bold,
                fontSize: 16)),
      ],
    );
  }
}


و

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../core/localization/localization_cubit.dart';
import '../../../core/localization/app_localizations.dart';

class LanguageSelectionScreen extends StatelessWidget {
  const LanguageSelectionScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final locale = context.watch<LocalizationCubit>().state.languageCode;
    const goldColor = Color(0xFFF5D142);

    return Scaffold(
      backgroundColor: const Color(0xFF101010),
      appBar: AppBar(
        title: Text(AppLocalizations.translate('choose_language', locale)),
        backgroundColor: const Color(0xFF101010),
        foregroundColor: Colors.white,
      ),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          children: [
            _langTile(
              context,
              'Français',
              'fr',
              locale == 'fr',
              goldColor,
            ),
            const SizedBox(height: 12),
            _langTile(
              context,
              'العربية',
              'ar',
              locale == 'ar',
              goldColor,
            ),
          ],
        ),
      ),
    );
  }

  Widget _langTile(BuildContext context, String title, String code, bool isSelected, Color goldColor) {
    return InkWell(
      onTap: () {
        context.read<LocalizationCubit>().changeLanguage(code);
      },
      borderRadius: BorderRadius.circular(12),
      child: Container(
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: const Color(0xFF141414),
          borderRadius: BorderRadius.circular(12),
          border: Border.all(
            color: isSelected ? goldColor : Colors.white10,
          ),
        ),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Text(
              title,
              style: TextStyle(
                color: isSelected ? goldColor : Colors.white,
                fontSize: 16,
                fontWeight: isSelected ? FontWeight.bold : FontWeight.normal,
              ),
            ),
            if (isSelected)
              Icon(Icons.check_circle, color: goldColor),
          ],
        ),
      ),
    );
  }
}


و

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../cubit/driver_cubit.dart';
import '../cubit/driver_state.dart';
import '../models/notification_model.dart';
import '../../../core/localization/app_localizations.dart';
import '../../../core/localization/localization_cubit.dart';
import '../../../core/theme/app_colors.dart';

class NotificationsScreen extends StatelessWidget {
  const NotificationsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final lang = context.watch<LocalizationCubit>().state.languageCode;
    const goldColor = Color(0xFFF5D142);

    return Scaffold(
      backgroundColor: AppColors.scaffold(context),
      appBar: AppBar(
        title: Text(AppLocalizations.translate('notifications', lang),
            style: const TextStyle(fontWeight: FontWeight.bold)),
        centerTitle: true,
        backgroundColor: AppColors.scaffold(context),
        foregroundColor: Colors.white,
        elevation: 0,
        actions: [
          IconButton(
            icon: const Icon(Icons.done_all, color: goldColor),
            onPressed: () {
              context.read<DriverCubit>().markAllNotificationsRead();
            },
            tooltip: 'Tout marquer comme lu',
          ),
        ],
      ),
      body: BlocBuilder<DriverCubit, DriverState>(
        builder: (context, state) {
          if (state is! DriverLoaded) {
            return const Center(child: CircularProgressIndicator(color: goldColor));
          }

          if (state.notifications.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.notifications_none, size: 64, color: Colors.grey.shade700),
                  const SizedBox(height: 16),
                  Text(
                    'Aucune notification',
                    style: TextStyle(color: Colors.grey.shade500, fontSize: 16),
                  ),
                ],
              ),
            );
          }

          return ListView.builder(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 10),
            itemCount: state.notifications.length,
            itemBuilder: (context, index) {
              final notification = state.notifications[index];
              return _NotificationCard(notification: notification, goldColor: goldColor);
            },
          );
        },
      ),
    );
  }
}

class _NotificationCard extends StatelessWidget {
  final NotificationModel notification;
  final Color goldColor;

  const _NotificationCard({required this.notification, required this.goldColor});

  @override
  Widget build(BuildContext context) {
    final color = _colorForType(notification.type);
    final icon = _iconForType(notification.type);

    return Container(
      margin: const EdgeInsets.only(bottom: 12),
      decoration: BoxDecoration(
        color: notification.isRead ? Colors.transparent : goldColor.withValues(alpha: 0.05),
        borderRadius: BorderRadius.circular(12),
        border: Border.all(
          color: notification.isRead ? Colors.white10 : goldColor.withValues(alpha: 0.3),
          width: 1,
        ),
      ),
      child: ListTile(
        onTap: () {
          if (!notification.isRead) {
            context.read<DriverCubit>().markNotificationRead(notification.id);
          }
        },
        contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        leading: Container(
          padding: const EdgeInsets.all(10),
          decoration: BoxDecoration(
            color: color.withValues(alpha: 0.1),
            shape: BoxShape.circle,
          ),
          child: Icon(icon, color: color, size: 22),
        ),
        title: Text(
          notification.title,
          style: TextStyle(
            color: Colors.white,
            fontWeight: notification.isRead ? FontWeight.normal : FontWeight.bold,
            fontSize: 15,
          ),
        ),
        subtitle: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const SizedBox(height: 4),
            Text(
              notification.message,
              style: TextStyle(color: Colors.grey.shade400, fontSize: 13),
            ),
            const SizedBox(height: 6),
            Text(
              _formatTime(notification.timestamp),
              style: TextStyle(color: Colors.grey.shade600, fontSize: 11),
            ),
          ],
        ),
        trailing: notification.isRead
            ? null
            : Container(
                width: 8,
                height: 8,
                decoration: const BoxDecoration(
                  color: Color(0xFFF5D142),
                  shape: BoxShape.circle,
                ),
              ),
      ),
    );
  }

  Color _colorForType(NotificationType type) {
    switch (type) {
      case NotificationType.newRequest: return goldColor;
      case NotificationType.tripCancelled: return Colors.redAccent;
      case NotificationType.statusChange: return Colors.blueAccent;
      case NotificationType.lowBalance: return Colors.orangeAccent;
      case NotificationType.tripCompleted: return Colors.greenAccent;
    }
  }

  IconData _iconForType(NotificationType type) {
    switch (type) {
      case NotificationType.newRequest: return Icons.notifications_active;
      case NotificationType.tripCancelled: return Icons.cancel;
      case NotificationType.statusChange: return Icons.swap_horiz;
      case NotificationType.lowBalance: return Icons.warning_amber;
      case NotificationType.tripCompleted: return Icons.check_circle;
    }
  }

  String _formatTime(DateTime timestamp) {
    final now = DateTime.now();
    final difference = now.difference(timestamp);

    if (difference.inMinutes < 60) {
      return 'Il y a ${difference.inMinutes} min';
    } else if (difference.inHours < 24) {
      return 'Il y a ${difference.inHours} h';
    } else {
      return '${timestamp.day}/${timestamp.month}';
    }
  }
}


و

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../cubit/driver_cubit.dart';
import '../cubit/driver_state.dart';
import '../../../core/localization/app_localizations.dart';
import '../../../core/localization/localization_cubit.dart';
import '../../../core/theme/app_colors.dart';

class PersonalInfoScreen extends StatefulWidget {
  final String driverId;

  const PersonalInfoScreen({super.key, required this.driverId});

  @override
  State<PersonalInfoScreen> createState() => _PersonalInfoScreenState();
}

class _PersonalInfoScreenState extends State<PersonalInfoScreen> {
  final _nameController = TextEditingController();
  final _phoneController = TextEditingController();
  bool _isLoading = false;

  @override
  void initState() {
    super.initState();
    final state = context.read<DriverCubit>().state;
    if (state is DriverLoaded) {
      _nameController.text = state.profile.name;
      _phoneController.text = state.profile.phone;
    }
  }

  @override
  void dispose() {
    _nameController.dispose();
    _phoneController.dispose();
    super.dispose();
  }

  void _save() async {
    final lang = context.read<LocalizationCubit>().state.languageCode;
    if (_nameController.text.isEmpty || _phoneController.text.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(AppLocalizations.translate('fill_all_fields', lang))),
      );
      return;
    }

    setState(() {
      _isLoading = true;
    });

    try {
      await context.read<DriverCubit>().updateProfile(
            widget.driverId,
            _nameController.text,
            _phoneController.text,
          );

      if (mounted) {
        setState(() {
          _isLoading = false;
        });
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text(AppLocalizations.translate('profile_updated', lang))),
        );
        Navigator.pop(context);
      }
    } catch (e) {
      if (mounted) {
        setState(() {
          _isLoading = false;
        });
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(e.toString().replaceAll('Exception: ', '')),
            backgroundColor: Colors.red,
          ),
        );
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    final lang = context.watch<LocalizationCubit>().state.languageCode;
    const goldColor = Color(0xFFF5D142);
    return Scaffold(
      backgroundColor: AppColors.scaffold(context),
      appBar: AppBar(
        title: const SizedBox.shrink(),
        centerTitle: true,
        backgroundColor: AppColors.scaffold(context),
        foregroundColor: Colors.white,
        elevation: 0,
      ),
      body: SingleChildScrollView(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 20),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // Profile Icon
              Center(
                child: Container(
                  padding: const EdgeInsets.all(20),
                  decoration: BoxDecoration(
                    color: goldColor.withValues(alpha: 0.1),
                    shape: BoxShape.circle,
                    border: Border.all(color: goldColor.withValues(alpha: 0.3), width: 2),
                  ),
                  child: const Icon(
                    Icons.person,
                    size: 80,
                    color: goldColor,
                  ),
                ),
              ),
              const SizedBox(height: 40),
              TextField(
                controller: _nameController,
                style: const TextStyle(color: Colors.white),
                decoration: InputDecoration(
                  labelText: AppLocalizations.translate('full_name', lang),
                  labelStyle: TextStyle(color: Colors.grey.shade500),
                  prefixIcon: const Icon(Icons.person_outline, color: goldColor),
                  enabledBorder: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                    borderSide: BorderSide(color: Colors.grey.shade800),
                  ),
                  focusedBorder: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                    borderSide: const BorderSide(color: goldColor),
                  ),
                  filled: true,
                  fillColor: AppColors.surface(context).withValues(alpha: 0.5),
                ),
              ),
              const SizedBox(height: 20),
              TextField(
                controller: _phoneController,
                style: const TextStyle(color: Colors.white),
                decoration: InputDecoration(
                  labelText: AppLocalizations.translate('phone', lang),
                  labelStyle: TextStyle(color: Colors.grey.shade500),
                  prefixIcon: const Icon(Icons.phone_outlined, color: goldColor),
                  enabledBorder: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                    borderSide: BorderSide(color: Colors.grey.shade800),
                  ),
                  focusedBorder: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                    borderSide: const BorderSide(color: goldColor),
                  ),
                  filled: true,
                  fillColor: AppColors.surface(context).withValues(alpha: 0.5),
                ),
                keyboardType: TextInputType.phone,
              ),
              const SizedBox(height: 40),
              ElevatedButton(
                style: ElevatedButton.styleFrom(
                  backgroundColor: goldColor,
                  foregroundColor: Colors.black,
                  padding: const EdgeInsets.symmetric(vertical: 18),
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                  elevation: 4,
                  shadowColor: goldColor.withValues(alpha: 0.3),
                ),
                onPressed: _isLoading ? null : _save,
                child: _isLoading
                    ? const SizedBox(
                        height: 22,
                        width: 22,
                        child: CircularProgressIndicator(color: Colors.black, strokeWidth: 2),
                      )
                    : Text(
                        AppLocalizations.translate('save', lang).toUpperCase(),
                        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold, letterSpacing: 1),
                      ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}


و

import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../../../core/localization/app_localizations.dart';
import '../../../core/localization/localization_cubit.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../core/theme/app_colors.dart';

class SecurityScreen extends StatefulWidget {
  const SecurityScreen({super.key});

  @override
  State<SecurityScreen> createState() => _SecurityScreenState();
}

class _SecurityScreenState extends State<SecurityScreen> {
  final _currentPasswordController = TextEditingController();
  final _newPasswordController = TextEditingController();
  final _confirmPasswordController = TextEditingController();
  bool _isLoading = false;
  bool _obscureCurrent = true;
  bool _obscureNew = true;
  bool _obscureConfirm = true;

  @override
  void dispose() {
    _currentPasswordController.dispose();
    _newPasswordController.dispose();
    _confirmPasswordController.dispose();
    super.dispose();
  }

  Future<void> _changePassword(String lang) async {
    final currentPassword = _currentPasswordController.text.trim();
    final newPassword = _newPasswordController.text.trim();
    final confirmPassword = _confirmPasswordController.text.trim();

    if (currentPassword.isEmpty || newPassword.isEmpty || confirmPassword.isEmpty) {
      _showSnackBar(AppLocalizations.translate('fill_all_fields', lang), Colors.redAccent);
      return;
    }
    if (newPassword != confirmPassword) {
      _showSnackBar(AppLocalizations.translate('passwords_not_match', lang), Colors.redAccent);
      return;
    }
    if (newPassword.length < 6) {
      _showSnackBar(AppLocalizations.translate('password_too_short', lang), Colors.orangeAccent);
      return;
    }

    setState(() => _isLoading = true);

    try {
      final user = FirebaseAuth.instance.currentUser;
      if (user == null || user.email == null) throw Exception('Non connecté');

      // Re-authenticate before changing password
      final credential = EmailAuthProvider.credential(
        email: user.email!,
        password: currentPassword,
      );
      await user.reauthenticateWithCredential(credential);
      await user.updatePassword(newPassword);

      _currentPasswordController.clear();
      _newPasswordController.clear();
      _confirmPasswordController.clear();
      _showSnackBar(AppLocalizations.translate('password_changed', lang), Colors.greenAccent);
    } on FirebaseAuthException catch (e) {
      String msg = AppLocalizations.translate('error_occurred', lang);
      if (e.code == 'wrong-password') {
        msg = AppLocalizations.translate('wrong_current_password', lang);
      } else if (e.code == 'requires-recent-login') {
        msg = AppLocalizations.translate('relogin_required', lang);
      }
      _showSnackBar(msg, Colors.redAccent);
    } catch (e) {
      _showSnackBar(e.toString(), Colors.redAccent);
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  void _showSnackBar(String msg, Color color) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(msg), backgroundColor: color, behavior: SnackBarBehavior.floating),
    );
  }

  @override
  Widget build(BuildContext context) {
    final lang = context.watch<LocalizationCubit>().state.languageCode;
    const goldColor = Color(0xFFF5D142);

    return Scaffold(
      backgroundColor: AppColors.scaffold(context),
      appBar: AppBar(
        backgroundColor: AppColors.scaffold(context),
        foregroundColor: Colors.white,
        elevation: 0,
        title: Text(AppLocalizations.translate('security', lang),
            style: const TextStyle(fontWeight: FontWeight.bold)),
        centerTitle: true,
        leading: IconButton(
          icon: const Icon(Icons.arrow_back_ios),
          onPressed: () => Navigator.pop(context),
        ),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Section Header
            _sectionHeader(Icons.lock_outline, AppLocalizations.translate('change_password', lang), goldColor),
            const SizedBox(height: 16),
            // Change Password Card
            Container(
              padding: const EdgeInsets.all(20),
              decoration: BoxDecoration(
                color: const Color(0xFF141414),
                borderRadius: BorderRadius.circular(16),
                border: Border.all(color: Colors.white10),
              ),
              child: Column(
                children: [
                  _passwordField(
                    controller: _currentPasswordController,
                    label: AppLocalizations.translate('current_password', lang),
                    obscure: _obscureCurrent,
                    onToggle: () => setState(() => _obscureCurrent = !_obscureCurrent),
                  ),
                  const SizedBox(height: 16),
                  _passwordField(
                    controller: _newPasswordController,
                    label: AppLocalizations.translate('new_password', lang),
                    obscure: _obscureNew,
                    onToggle: () => setState(() => _obscureNew = !_obscureNew),
                  ),
                  const SizedBox(height: 16),
                  _passwordField(
                    controller: _confirmPasswordController,
                    label: AppLocalizations.translate('confirm_password', lang),
                    obscure: _obscureConfirm,
                    onToggle: () => setState(() => _obscureConfirm = !_obscureConfirm),
                  ),
                  const SizedBox(height: 24),
                  SizedBox(
                    width: double.infinity,
                    height: 50,
                    child: ElevatedButton(
                      style: ElevatedButton.styleFrom(
                        backgroundColor: goldColor,
                        foregroundColor: Colors.black,
                        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                      ),
                      onPressed: _isLoading ? null : () => _changePassword(lang),
                      child: _isLoading
                          ? const SizedBox(height: 20, width: 20, child: CircularProgressIndicator(color: Colors.black, strokeWidth: 2))
                          : Text(AppLocalizations.translate('save', lang),
                              style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 28),
            // Security Tips Section
            _sectionHeader(Icons.shield_outlined, AppLocalizations.translate('security_tips', lang), Colors.blueAccent),
            const SizedBox(height: 16),
            Container(
              padding: const EdgeInsets.all(20),
              decoration: BoxDecoration(
                color: const Color(0xFF141414),
                borderRadius: BorderRadius.circular(16),
                border: Border.all(color: Colors.white10),
              ),
              child: Column(
                children: [
                  _tip(Icons.check_circle_outline, AppLocalizations.translate('tip_strong_password', lang)),
                  _tip(Icons.check_circle_outline, AppLocalizations.translate('tip_dont_share', lang)),
                  _tip(Icons.check_circle_outline, AppLocalizations.translate('tip_logout_public', lang)),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _sectionHeader(IconData icon, String title, Color color) {
    return Row(
      children: [
        Icon(icon, color: color, size: 22),
        const SizedBox(width: 10),
        Text(title, style: TextStyle(color: color, fontSize: 17, fontWeight: FontWeight.bold)),
      ],
    );
  }

  Widget _passwordField({
    required TextEditingController controller,
    required String label,
    required bool obscure,
    required VoidCallback onToggle,
  }) {
    return TextField(
      controller: controller,
      obscureText: obscure,
      style: const TextStyle(color: Colors.white),
      decoration: InputDecoration(
        labelText: label,
        labelStyle: TextStyle(color: Colors.grey.shade500),
        enabledBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(10),
          borderSide: BorderSide(color: Colors.grey.shade800),
        ),
        focusedBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(10),
          borderSide: const BorderSide(color: Color(0xFFF5D142)),
        ),
        suffixIcon: IconButton(
          icon: Icon(obscure ? Icons.visibility_off : Icons.visibility, color: Colors.grey),
          onPressed: onToggle,
        ),
      ),
    );
  }

  Widget _tip(IconData icon, String text) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 12),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Icon(icon, color: Colors.greenAccent, size: 18),
          const SizedBox(width: 10),
          Expanded(child: Text(text, style: TextStyle(color: Colors.grey.shade400, fontSize: 13))),
        ],
      ),
    );
  }
}


و

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../cubit/driver_cubit.dart';
import '../cubit/driver_state.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../widgets/driver_status_badge.dart';
import 'personal_info_screen.dart';
import 'language_selection_screen.dart';
import 'security_screen.dart';
import 'help_support_screen.dart';
import 'about_screen.dart';
import 'notifications_screen.dart';
import '../../../core/localization/app_localizations.dart';
import '../../../core/localization/localization_cubit.dart';
import '../../../core/theme/app_colors.dart';

class SettingsScreen extends StatelessWidget {
  final String driverId;

  const SettingsScreen({super.key, required this.driverId});

  @override
  Widget build(BuildContext context) {
    final lang = context.watch<LocalizationCubit>().state.languageCode;
    const goldColor = Color(0xFFF5D142);

    return Scaffold(
      backgroundColor: AppColors.scaffold(context),
      appBar: AppBar(
        backgroundColor: AppColors.scaffold(context),
        foregroundColor: Colors.white,
        elevation: 0,
        title: Text(AppLocalizations.translate('settings', lang),
            style: const TextStyle(fontWeight: FontWeight.bold)),
        centerTitle: true,
        leading: IconButton(
          icon: const Icon(Icons.arrow_back_ios),
          onPressed: () => Navigator.pop(context),
        ),
      ),
      body: BlocBuilder<DriverCubit, DriverState>(
        builder: (context, state) {
          if (state is! DriverLoaded) {
            return const Center(
              child: CircularProgressIndicator(color: goldColor),
            );
          }

          final profile = state.profile;
          final isOnline = profile.status == 'online' ||
              profile.status == 'available';

          return SingleChildScrollView(
            padding: const EdgeInsets.all(20),
            child: Column(
              children: [
                // Profile card
                Container(
                  width: double.infinity,
                  padding: const EdgeInsets.all(24),
                  decoration: BoxDecoration(
                    color: AppColors.surface(context),
                    borderRadius: BorderRadius.circular(16),
                    border: Border.all(color: goldColor.withValues(alpha: 0.3)),
                  ),
                  child: Column(
                    children: [
                      CircleAvatar(
                        radius: 40,
                        backgroundColor: goldColor,
                        child: Text(
                          profile.name.isNotEmpty
                              ? profile.name[0].toUpperCase()
                              : 'D',
                          style: const TextStyle(
                            color: Colors.black,
                            fontSize: 32,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ),
                      const SizedBox(height: 14),
                      Text(profile.name,
                          style: TextStyle(
                              color: AppColors.onSurface(context),
                              fontSize: 20,
                              fontWeight: FontWeight.bold)),
                      if (profile.phone.isNotEmpty) ...[
                        const SizedBox(height: 4),
                        Text(profile.phone,
                            style: TextStyle(
                                color: Colors.grey.shade500, fontSize: 14)),
                      ],
                      const SizedBox(height: 12),
                      DriverStatusBadge(status: profile.status),
                      const SizedBox(height: 8),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          Text('${profile.rating.toStringAsFixed(1)} ⭐',
                              style: TextStyle(
                                  color: AppColors.onSurface(context), fontSize: 14)),
                          const SizedBox(width: 16),
                          Text('${profile.totalTrips} ${AppLocalizations.translate('trips', lang)}',
                              style: TextStyle(
                                  color: Colors.grey.shade400, fontSize: 14)),
                        ],
                      ),
                    ],
                  ),
                ),
                const SizedBox(height: 24),
                // Online/Offline toggle
                Container(
                  padding: const EdgeInsets.all(16),
                  decoration: BoxDecoration(
                    color: AppColors.surface(context),
                    borderRadius: BorderRadius.circular(14),
                    border: Border.all(color: AppColors.border(context)),
                  ),
                  child: Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Row(
                        children: [
                          const Icon(Icons.power_settings_new,
                              color: Colors.white, size: 22),
                          const SizedBox(width: 12),
                          const Text('Disponibilité',
                              style: TextStyle(
                                  color: Colors.white, fontSize: 16)),
                        ],
                      ),
                      Switch(
                        value: isOnline,
                        activeThumbColor: goldColor,
                        onChanged: (value) {
                          context
                              .read<DriverCubit>()
                              .toggleStatus(driverId, value);
                        },
                      ),
                    ],
                  ),
                ),
                const SizedBox(height: 16),
                // Settings items
                _settingsTile(
                  context: context,
                  icon: Icons.person_outline,
                  label: AppLocalizations.translate('personal_info', lang),
                  onTap: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (_) => BlocProvider.value(
                          value: context.read<DriverCubit>(),
                          child: PersonalInfoScreen(driverId: driverId),
                        ),
                      ),
                    );
                  },
                ),
                _settingsTile(
                  context: context,
                  icon: Icons.notifications_outlined,
                  label: AppLocalizations.translate('notifications', lang),
                  trailing: Text(
                    '${state.unreadNotificationCount}',
                    style: const TextStyle(color: goldColor),
                  ),
                  onTap: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (_) => BlocProvider.value(
                          value: context.read<DriverCubit>(),
                          child: const NotificationsScreen(),
                        ),
                      ),
                    );
                  },
                ),
                _settingsTile(
                  context: context,
                  icon: Icons.security_outlined,
                  label: AppLocalizations.translate('security', lang),
                  onTap: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(builder: (_) => const SecurityScreen()),
                    );
                  },
                ),
                _settingsTile(
                  context: context,
                  icon: Icons.language,
                  label: AppLocalizations.translate('language', lang),
                  trailing: Text(
                    AppLocalizations.translate(lang == 'fr' ? 'french' : 'arabic', lang),
                    style: TextStyle(color: Colors.grey.shade500),
                  ),
                  onTap: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (ctx) => const LanguageSelectionScreen(),
                      ),
                    );
                  },
                ),
                _settingsTile(
                  context: context,
                  icon: Icons.help_outline,
                  label: AppLocalizations.translate('help_support', lang),
                  onTap: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(builder: (_) => const HelpSupportScreen()),
                    );
                  },
                ),
                _settingsTile(
                  context: context,
                  icon: Icons.info_outline,
                  label: AppLocalizations.translate('about', lang),
                  onTap: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(builder: (_) => const AboutScreen()),
                    );
                  },
                ),
                const SizedBox(height: 24),
                // Logout
                SizedBox(
                  width: double.infinity,
                  height: 50,
                  child: OutlinedButton.icon(
                    style: OutlinedButton.styleFrom(
                      foregroundColor: Colors.redAccent,
                      side: const BorderSide(color: Colors.redAccent),
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(12),
                      ),
                    ),
                    icon: const Icon(Icons.logout),
                    label: Text(AppLocalizations.translate('logout', lang),
                        style: const TextStyle(fontWeight: FontWeight.bold)),
                    onPressed: () async {
                      await FirebaseAuth.instance.signOut();
                      if (context.mounted) {
                        Navigator.of(context).pushNamedAndRemoveUntil('/', (route) => false);
                      }
                    },
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }

  Widget _settingsTile({
    required BuildContext context,
    required IconData icon,
    required String label,
    Widget? trailing,
    required VoidCallback onTap,
  }) {
    return Container(
      margin: const EdgeInsets.only(bottom: 8),
      decoration: BoxDecoration(
        color: AppColors.surface(context),
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: Colors.white10),
      ),
      child: ListTile(
        leading: Icon(icon, color: Colors.white70, size: 22),
        title: Text(label,
            style: TextStyle(color: AppColors.onSurface(context), fontSize: 15)),
        trailing: trailing ??
            Icon(Icons.chevron_right, color: Colors.grey.shade700),
        onTap: onTap,
      ),
    );
  }
}


و

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../cubit/driver_cubit.dart';
import '../cubit/driver_state.dart';
import '../widgets/map_component.dart';
import 'package:latlong2/latlong.dart' as ll;
import 'package:flutter_map/flutter_map.dart' as fm;
import '../widgets/trip_info_panel.dart';
import '../../../core/localization/app_localizations.dart';
import '../../../core/localization/localization_cubit.dart';
import 'package:url_launcher/url_launcher.dart';
import '../../../core/theme/app_colors.dart';

class TripViewScreen extends StatelessWidget {
  final String tripId;
  final String driverId;

  const TripViewScreen({
    super.key,
    required this.tripId,
    required this.driverId,
  });

  @override
  Widget build(BuildContext context) {
    final lang = context.watch<LocalizationCubit>().state.languageCode;
    const goldColor = Color(0xFFF5D142);

    return Scaffold(
      backgroundColor: AppColors.scaffold(context),
      body: SafeArea(
        child: BlocBuilder<DriverCubit, DriverState>(
          builder: (context, state) {
            if (state is! DriverLoaded || state.activeTrip == null) {
              return Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    const Icon(Icons.check_circle,
                        size: 64, color: Colors.greenAccent),
                    const SizedBox(height: 16),
                    Text(AppLocalizations.translate('trip_completed', lang),
                        style: const TextStyle(
                            color: Colors.white,
                            fontSize: 20,
                            fontWeight: FontWeight.bold)),
                    const SizedBox(height: 24),
                    ElevatedButton(
                      style: ElevatedButton.styleFrom(
                        backgroundColor: goldColor,
                        foregroundColor: Colors.black,
                        padding: const EdgeInsets.symmetric(
                            horizontal: 40, vertical: 14),
                      ),
                      onPressed: () => Navigator.of(context).pop(),
                      child: Text(AppLocalizations.translate('back', lang),
                          style: const TextStyle(fontWeight: FontWeight.bold)),
                    ),
                  ],
                ),
              );
            }

            final trip = state.activeTrip!;
            final tripStatus = trip.status;

            return Column(
              children: [
                // Top bar
                Padding(
                  padding: const EdgeInsets.symmetric(
                      horizontal: 16, vertical: 8),
                  child: Row(
                    children: [
                      IconButton(
                        icon: const Icon(Icons.arrow_back_ios,
                            color: Colors.white),
                        onPressed: () => Navigator.of(context).pop(),
                      ),
                      Expanded(
                        child: Text(
                          AppLocalizations.translate('navigation', lang),
                          textAlign: TextAlign.center,
                          style: const TextStyle(
                            color: Colors.white,
                            fontSize: 20,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ),
                      const SizedBox(width: 48), // balance the back button
                    ],
                  ),
                ),
                // Map area
                Expanded(
                  flex: 3,
                  child: MapComponent(
                    routePoints: state.routePoints,
                    pickupLatLng: (trip.pickupLat != null && trip.pickupLng != null)
                        ? ll.LatLng(trip.pickupLat!, trip.pickupLng!)
                        : null,
                    destinationLatLng: (trip.destinationLat != null && trip.destinationLng != null)
                        ? ll.LatLng(trip.destinationLat!, trip.destinationLng!)
                        : null,
                    markers: [
                      // Pickup Location Marker (Green)
                      if (trip.pickupLat != null && trip.pickupLng != null)
                        fm.Marker(
                          point: ll.LatLng(trip.pickupLat!, trip.pickupLng!),
                          width: 50,
                          height: 50,
                          child: Column(
                            mainAxisSize: MainAxisSize.min,
                            children: [
                              const Icon(Icons.location_on, color: Colors.greenAccent, size: 36),
                              Text(AppLocalizations.translate('pickup_point', lang), style: const TextStyle(color: Colors.greenAccent, fontSize: 9, fontWeight: FontWeight.bold)),
                            ],
                          ),
                        ),
                      // Destination Location Marker (Red)
                      if (trip.destinationLat != null && trip.destinationLng != null)
                        fm.Marker(
                          point: ll.LatLng(trip.destinationLat!, trip.destinationLng!),
                          width: 50,
                          height: 50,
                          child: Column(
                            mainAxisSize: MainAxisSize.min,
                            children: [
                              const Icon(Icons.flag_circle, color: Colors.redAccent, size: 36),
                              Text(AppLocalizations.translate('destination_point', lang), style: const TextStyle(color: Colors.redAccent, fontSize: 9, fontWeight: FontWeight.bold)),
                            ],
                          ),
                        ),
                      // Driver Location Marker (Gold)
                      if (state.profile.latitude != null && state.profile.longitude != null)
                        fm.Marker(
                          point: ll.LatLng(state.profile.latitude!, state.profile.longitude!),
                          width: 44,
                          height: 44,
                          child: Container(
                            decoration: BoxDecoration(
                              color: Colors.black,
                              shape: BoxShape.circle,
                              border: Border.all(color: goldColor, width: 2),
                              boxShadow: [
                                BoxShadow(
                                  color: goldColor.withValues(alpha: 0.4),
                                  blurRadius: 8,
                                  spreadRadius: 2,
                                ),
                              ],
                            ),
                            child: const Icon(Icons.local_taxi, color: Color(0xFFF5D142), size: 24),
                          ),
                        ),
                    ],
                  ),
                ),
                // Trip info panel
                TripInfoPanel(trip: trip),
                // Action buttons
                _buildActionButtons(context, trip, goldColor, lang),
                const SizedBox(height: 16),
              ],
            );
          },
        ),
      ),
    );
  }

  Widget _buildActionButtons(
      BuildContext context, dynamic trip, Color goldColor, String lang) {
    final status = trip.status;
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Column(
        children: [
          // Step 1: Call Client & Cancel Button
          Row(
            children: [
              Expanded(
                child: _actionButton(
                  icon: Icons.call,
                  label: AppLocalizations.translate('call_client', lang),
                  color: Colors.white,
                  bgColor: const Color(0xFF1A1A1A),
                  borderColor: Colors.white24,
                  onPressed: () async {
                    final phone = trip.clientPhone as String;
                    if (phone.isNotEmpty) {
                      final Uri url = Uri.parse('tel:$phone');
                      try {
                        if (await canLaunchUrl(url)) {
                          await launchUrl(url);
                        } else {
                          if (context.mounted) {
                            ScaffoldMessenger.of(context).showSnackBar(
                              SnackBar(
                                content: Text('Impossible de lancer l\'appel au $phone'),
                                backgroundColor: Colors.redAccent,
                              ),
                            );
                          }
                        }
                      } catch (e) {
                        if (context.mounted) {
                          ScaffoldMessenger.of(context).showSnackBar(
                            SnackBar(
                              content: Text('Erreur: $e'),
                              backgroundColor: Colors.redAccent,
                            ),
                          );
                        }
                      }
                    } else {
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          content: Text('Numéro de téléphone non disponible'),
                          backgroundColor: Colors.redAccent,
                        ),
                      );
                    }
                  },
                ),
              ),
              const SizedBox(width: 12),
              // Cancel Button
              Expanded(
                child: _actionButton(
                  icon: Icons.close,
                  label: AppLocalizations.translate('cancel', lang),
                  color: Colors.redAccent,
                  bgColor: Colors.red.withValues(alpha: 0.1),
                  borderColor: Colors.redAccent.withValues(alpha: 0.3),
                  onPressed: () {
                    _showCancelDialog(context, lang);
                  },
                ),
              ),
            ],
          ),
          const SizedBox(height: 12),
          // Step 2-4: Sequential action (contextual)
          SizedBox(
            width: double.infinity,
            height: 56,
            child: ElevatedButton.icon(
              style: ElevatedButton.styleFrom(
                backgroundColor: _getMainButtonColor(status, goldColor),
                foregroundColor: Colors.black,
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(12),
                ),
              ),
              icon: Icon(_getMainButtonIcon(status)),
              label: Text(
                _getMainButtonLabel(status, lang),
                style: const TextStyle(
                  fontWeight: FontWeight.bold,
                  fontSize: 16,
                  letterSpacing: 0.5,
                ),
              ),
              onPressed: () => _handleMainAction(context, status, lang),
            ),
          ),
        ],
      ),
    );
  }

  Widget _actionButton({
    required IconData icon,
    required String label,
    required Color color,
    required Color bgColor,
    required Color borderColor,
    required VoidCallback onPressed,
  }) {
    return SizedBox(
      height: 50,
      child: OutlinedButton.icon(
        style: OutlinedButton.styleFrom(
          foregroundColor: color,
          backgroundColor: bgColor,
          side: BorderSide(color: borderColor),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(10),
          ),
        ),
        icon: Icon(icon, size: 18),
        label: Text(label, style: const TextStyle(fontSize: 13)),
        onPressed: onPressed,
      ),
    );
  }

  String _getMainButtonLabel(String status, String lang) {
    switch (status) {
      case 'accepted':
        return AppLocalizations.translate('arrived_at_point', lang);
      case 'arrived':
        return AppLocalizations.translate('start_trip', lang);
      case 'in_progress':
        return AppLocalizations.translate('finish_trip', lang);
      default:
        return AppLocalizations.translate('waiting', lang);
    }
  }

  IconData _getMainButtonIcon(String status) {
    switch (status) {
      case 'accepted':
        return Icons.pin_drop;
      case 'arrived':
        return Icons.play_arrow;
      case 'in_progress':
        return Icons.flag;
      default:
        return Icons.hourglass_empty;
    }
  }

  Color _getMainButtonColor(String status, Color goldColor) {
    switch (status) {
      case 'accepted':
        return Colors.orangeAccent;
      case 'arrived':
        return Colors.greenAccent;
      case 'in_progress':
        return goldColor;
      default:
        return Colors.grey;
    }
  }

  void _handleMainAction(BuildContext context, String status, String lang) {
    final cubit = context.read<DriverCubit>();
    switch (status) {
      case 'accepted':
        cubit.arrivedAtPickup(tripId);
        break;
      case 'arrived':
        cubit.startTrip(tripId, driverId);
        break;
      case 'in_progress':
        _showEndTripDialog(context, lang);
        break;
    }
  }

  void _showEndTripDialog(BuildContext context, String lang) {
    const goldColor = Color(0xFFF5D142);
    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        backgroundColor: const Color(0xFF1A1A1A),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
        title: Text(AppLocalizations.translate('finish_trip_q', lang),
            style: const TextStyle(color: Colors.white)),
        content: Text(
          AppLocalizations.translate('confirm_finish_trip_msg', lang),
          style: const TextStyle(color: Colors.grey),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx),
            child: Text(AppLocalizations.translate('cancel', lang), style: const TextStyle(color: Colors.grey)),
          ),
          ElevatedButton(
            style: ElevatedButton.styleFrom(
                backgroundColor: goldColor, foregroundColor: Colors.black),
            onPressed: () {
              Navigator.pop(ctx);
              // Use estimated price as fare with 15% default commission
              final state = context.read<DriverCubit>().state;
              if (state is DriverLoaded && state.activeTrip != null) {
                final fare = state.activeTrip!.estimatedPrice;
                final commissionRate = state.profile.commissionRate;
                context.read<DriverCubit>().endTrip(
                    tripId, driverId, fare, commissionRate);
              }
            },
            child: Text(AppLocalizations.translate('confirm', lang),
                style: const TextStyle(fontWeight: FontWeight.bold)),
          ),
        ],
      ),
    );
  }

  void _showCancelDialog(BuildContext context, String lang) {
    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        backgroundColor: const Color(0xFF1A1A1A),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
        title: Text(AppLocalizations.translate('cancel_trip_q', lang),
            style: const TextStyle(color: Colors.white)),
        content: Text(
          AppLocalizations.translate('confirm_cancel_trip_msg', lang),
          style: const TextStyle(color: Colors.grey),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx),
            child: Text(AppLocalizations.translate('no', lang), style: const TextStyle(color: Colors.grey)),
          ),
          ElevatedButton(
            style: ElevatedButton.styleFrom(
                backgroundColor: Colors.redAccent,
                foregroundColor: Colors.white),
            onPressed: () {
              Navigator.pop(ctx);
              context
                  .read<DriverCubit>()
                  .cancelTrip(tripId, driverId);
              Navigator.of(context).pop(); // go back to home
            },
            child: Text(AppLocalizations.translate('yes_cancel', lang),
                style: const TextStyle(fontWeight: FontWeight.bold)),
          ),
        ],
      ),
    );
  }
}


و


import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../cubit/driver_cubit.dart';
import '../cubit/driver_state.dart';
import 'history_screen.dart';
import '../../../core/localization/app_localizations.dart';
import '../../../core/localization/localization_cubit.dart';
import '../../../core/theme/app_colors.dart';

class WalletScreen extends StatelessWidget {
  final String driverId;

  const WalletScreen({super.key, required this.driverId});

  @override
  Widget build(BuildContext context) {
    final lang = context.watch<LocalizationCubit>().state.languageCode;
    const goldColor = Color(0xFFF5D142);

    return Scaffold(
      backgroundColor: AppColors.scaffold(context),
      appBar: AppBar(
        backgroundColor: AppColors.scaffold(context),
        foregroundColor: Colors.white,
        elevation: 0,
        title: Text(AppLocalizations.translate('my_wallet', lang),
            style: const TextStyle(fontWeight: FontWeight.bold)),
        centerTitle: true,
        leading: IconButton(
          icon: const Icon(Icons.arrow_back_ios),
          onPressed: () => Navigator.pop(context),
        ),
      ),
      body: BlocBuilder<DriverCubit, DriverState>(
        builder: (context, state) {
          if (state is! DriverLoaded) {
            return const Center(
              child: CircularProgressIndicator(color: goldColor),
            );
          }

          final profile = state.profile;
          final history = state.tripHistory;

          // Calculate stats from history
          double totalEarned = 0;
          double totalCommission = 0;
          for (final trip in history) {
            totalEarned += (trip.fare ?? trip.estimatedPrice);
            totalCommission += (trip.commission ?? 0);
          }

          return SingleChildScrollView(
            padding: const EdgeInsets.all(20),
            child: Column(
              children: [
                // Balance card
                Container(
                  width: double.infinity,
                  padding: const EdgeInsets.all(28),
                  decoration: BoxDecoration(
                    gradient: LinearGradient(
                      begin: Alignment.topLeft,
                      end: Alignment.bottomRight,
                      colors: [
                        const Color(0xFF1A1A1A),
                        const Color(0xFF141414),
                        Colors.black.withValues(alpha: 0.9),
                      ],
                    ),
                    borderRadius: BorderRadius.circular(20),
                    border: Border.all(color: goldColor.withValues(alpha: 0.4)),
                    boxShadow: [
                      BoxShadow(
                        color: goldColor.withValues(alpha: 0.1),
                        blurRadius: 20,
                        spreadRadius: 2,
                      ),
                    ],
                  ),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text(AppLocalizations.translate('current_balance', lang),
                              style:
                                  const TextStyle(color: Colors.grey, fontSize: 14)),
                          Icon(Icons.account_balance_wallet,
                              color: goldColor.withValues(alpha: 0.6), size: 28),
                        ],
                      ),
                      const SizedBox(height: 12),
                      Text(
                        '${profile.walletBalance.toStringAsFixed(2)} MRU',
                        style: const TextStyle(
                          color: goldColor,
                          fontSize: 42,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 20),
                      // Low balance warning
                      if (profile.isLowBalance)
                        Container(
                          padding: const EdgeInsets.all(10),
                          decoration: BoxDecoration(
                            color: Colors.orange.withValues(alpha: 0.1),
                            borderRadius: BorderRadius.circular(8),
                            border: Border.all(
                                color: Colors.orangeAccent.withValues(alpha: 0.3)),
                          ),
                          child: Row(
                            children: [
                              const Icon(Icons.warning_amber,
                                  color: Colors.orangeAccent, size: 18),
                              const SizedBox(width: 8),
                              Expanded(
                                child: Text(
                                  AppLocalizations.translate('insufficient_balance_recharge_msg', lang),
                                  style: const TextStyle(
                                      color: Colors.orangeAccent, fontSize: 12),
                                ),
                              ),
                            ],
                          ),
                        ),
                      const SizedBox(height: 20),
                      Row(
                        children: [
                          Expanded(
                            child: ElevatedButton.icon(
                              style: ElevatedButton.styleFrom(
                                backgroundColor: Colors.transparent,
                                foregroundColor: goldColor,
                                side: const BorderSide(color: goldColor),
                                padding:
                                    const EdgeInsets.symmetric(vertical: 14),
                                shape: RoundedRectangleBorder(
                                  borderRadius: BorderRadius.circular(10),
                                ),
                              ),
                              icon: const Icon(Icons.history),
                              label: Text(AppLocalizations.translate('history', lang)),
                              onPressed: () {
                                Navigator.of(context).push(
                                  MaterialPageRoute(
                                    builder: (_) => BlocProvider.value(
                                      value: context.read<DriverCubit>(),
                                      child: HistoryScreen(driverId: driverId),
                                    ),
                                  ),
                                );
                              },
                            ),
                          ),
                          const SizedBox(width: 16),
                          Expanded(
                            child: ElevatedButton.icon(
                              style: ElevatedButton.styleFrom(
                                backgroundColor: goldColor,
                                foregroundColor: Colors.black,
                                padding:
                                    const EdgeInsets.symmetric(vertical: 14),
                                shape: RoundedRectangleBorder(
                                  borderRadius: BorderRadius.circular(10),
                                ),
                              ),
                              icon: const Icon(Icons.arrow_downward),
                              label: Text(AppLocalizations.translate('withdraw', lang)),
                              onPressed: () {},
                            ),
                          ),
                        ],
                      ),
                    ],
                  ),
                ),
                const SizedBox(height: 28),
                // Stats
                Row(
                  children: [
                    Expanded(
                        child: _statCard(
                            context,
                            AppLocalizations.translate('total_earned', lang),
                            '${totalEarned.toStringAsFixed(2)} MRU',
                            Icons.trending_up,
                            Colors.greenAccent)),
                    const SizedBox(width: 16),
                    Expanded(
                        child: _statCard(
                            context,
                            AppLocalizations.translate('commission_label', lang),
                            '${totalCommission.toStringAsFixed(2)} MRU',
                            Icons.percent,
                            Colors.redAccent)),
                  ],
                ),
                const SizedBox(height: 16),
                Row(
                  children: [
                    Expanded(
                        child: _statCard(
                            context,
                            AppLocalizations.translate('trips', lang),
                            '${profile.totalTrips}',
                            Icons.local_taxi,
                            goldColor)),
                    const SizedBox(width: 16),
                    Expanded(
                        child: _statCard(
                            context,
                            AppLocalizations.translate('rating_label', lang),
                            '${profile.rating.toStringAsFixed(1)} ⭐',
                            Icons.star,
                            Colors.orangeAccent)),
                  ],
                ),
                const SizedBox(height: 28),
                // Transaction history
                Align(
                  alignment: Alignment.centerLeft,
                  child: Text(
                    AppLocalizations.translate('recent_transactions', lang),
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
                const SizedBox(height: 12),
                if (history.isEmpty)
                  Padding(
                    padding: const EdgeInsets.only(top: 30),
                    child: Text(AppLocalizations.translate('no_transactions', lang),
                        style: TextStyle(color: Colors.grey.shade600)),
                  )
                else
                  ...history.take(10).map((trip) => _transactionTile(context, trip)),
              ],
            ),
          );
        },
      ),
    );
  }

  Widget _statCard(
      BuildContext context, String label, String value, IconData icon, Color color) {
    return Container(
      padding: const EdgeInsets.all(18),
      decoration: BoxDecoration(
        color: AppColors.surface(context),
        borderRadius: BorderRadius.circular(14),
        border: Border.all(color: Colors.white10),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Icon(icon, color: color, size: 22),
          const SizedBox(height: 10),
          Text(value,
              style: TextStyle(
                  color: color, fontWeight: FontWeight.bold, fontSize: 18)),
          const SizedBox(height: 2),
          Text(label,
              style: TextStyle(color: Colors.grey.shade500, fontSize: 12)),
        ],
      ),
    );
  }

  Widget _transactionTile(BuildContext context, dynamic trip) {
    final fare = (trip.fare ?? trip.estimatedPrice) as double;
    final commission = (trip.commission ?? 0.0) as double;
    final net = fare - commission;

    return Container(
      margin: const EdgeInsets.only(bottom: 8),
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: AppColors.surface(context),
        borderRadius: BorderRadius.circular(10),
        border: Border.all(color: Colors.white10),
      ),
      child: Row(
        children: [
          Container(
            padding: const EdgeInsets.all(8),
            decoration: BoxDecoration(
              color: Colors.greenAccent.withValues(alpha: 0.1),
              borderRadius: BorderRadius.circular(8),
            ),
            child: const Icon(Icons.check_circle,
                color: Colors.greenAccent, size: 20),
          ),
          const SizedBox(width: 14),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(trip.clientName as String,
                    style: const TextStyle(
                        color: Colors.white, fontWeight: FontWeight.bold)),
                Text(
                  '${trip.distance.toStringAsFixed(1)} km',
                  style: TextStyle(color: Colors.grey.shade500, fontSize: 12),
                ),
              ],
            ),
          ),
          Text('+${net.toStringAsFixed(2)} MRU',
              style: const TextStyle(
                  color: Colors.greenAccent,
                  fontWeight: FontWeight.bold,
                  fontSize: 15)),
        ],
      ),
    );
  }
}


و

import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../models/driver.dart';

class DriverService {
  final FirebaseFirestore _firestore;

  DriverService({FirebaseFirestore? firestore})
      : _firestore = firestore ?? FirebaseFirestore.instance;

  /// Stream driver profile in real-time from 'drivers' collection
  Stream<DriverModel> streamDriverModel(String driverId) {
    return _firestore.collection('drivers').doc(driverId).snapshots().map(
      (snapshot) {
        if (!snapshot.exists || snapshot.data() == null) {
          throw Exception('Driver profile not found');
        }
        return DriverModel.fromJson(snapshot.data()!, snapshot.id);
      },
    );
  }

  String _normalizeStatus(String status) {
    final s = status.toLowerCase().trim();
    if (s == 'active' || s == 'available') return 'online';
    if (s == 'busy' || s == 'on_trip') return 'busy';
    if (s == 'online') return 'online';
    return 'offline';
  }

  /// Stream wallet balance from 'wallets' collection
  Stream<double> streamWalletBalance(String driverId) {
    return _firestore.collection('wallets').doc(driverId).snapshots().map(
      (snapshot) {
        if (!snapshot.exists || snapshot.data() == null) {
          return 0.0;
        }
        final dynamic val = snapshot.data()!['balance'];
        if (val is num) return val.toDouble();
        if (val is String) return double.tryParse(val) ?? 0.0;
        return 0.0;
      },
    );
  }

  /// Get wallet balance once (for initial load)
  Future<double> getWalletBalance(String driverId) async {
    try {
      final doc = await _firestore.collection('wallets').doc(driverId).get();
      if (doc.exists && doc.data() != null) {
        final dynamic val = doc.data()!['balance'];
        if (val is num) return val.toDouble();
        if (val is String) return double.tryParse(val) ?? 0.0;
      }
    } catch (_) {}
    return 0.0;
  }

  /// Update driver status
  Future<void> updateDriverStatus(String driverId, String status) async {
    final normalized = _normalizeStatus(status);
    final online = normalized == 'online';
    try {
      await _firestore.collection('drivers').doc(driverId).update({
        'status': normalized,
        'isOnline': online,
        'is_online': online,
      });
    } catch (_) {
      // Document may not exist yet; set it
      await _firestore.collection('drivers').doc(driverId).set({
        'status': normalized,
        'isOnline': online,
        'is_online': online,
      }, SetOptions(merge: true));
    }
  }

  /// Go online
  Future<void> goOnline(String driverId) =>
      updateDriverStatus(driverId, 'online');

  /// Go offline
  Future<void> goOffline(String driverId) =>
      updateDriverStatus(driverId, 'offline');

  /// Block driver (insufficient balance or admin action)
  Future<void> blockDriver(String driverId) =>
      updateDriverStatus(driverId, 'blocked');

  /// Unblock driver
  Future<void> unblockDriver(String driverId) =>
      updateDriverStatus(driverId, 'offline');

  /// Check and create wallet document if missing (for old drivers)
  Future<void> ensureWalletExists(String driverId, double fallbackBalance) async {
    try {
      final doc = await _firestore.collection('wallets').doc(driverId).get();
      if (!doc.exists) {
        await _firestore.collection('wallets').doc(driverId).set({
          'balance': fallbackBalance,
        });
      }
    } catch (_) {}
  }

  /// Update wallet balance (add or subtract) in 'wallets' collection
  Future<void> updateWalletBalance(String driverId, double amount) async {
    try {
      await _firestore.collection('wallets').doc(driverId).update({
        'balance': FieldValue.increment(amount),
      });
    } catch (_) {
      await _firestore.collection('wallets').doc(driverId).set({
        'balance': amount,
      }, SetOptions(merge: true));
    }
  }

  /// Increment total trips
  Future<void> incrementTotalTrips(String driverId) async {
    try {
      await _firestore.collection('drivers').doc(driverId).update({
        'totalTrips': FieldValue.increment(1),
      });
    } catch (_) {
      // ignore
    }
  }

  /// Check balance and block if too low (checks 'wallets' collection)
  Future<bool> checkAndBlockIfLowBalance(String driverId) async {
    try {
      final doc = await _firestore.collection('wallets').doc(driverId).get();
      if (doc.exists) {
        final balance = (doc.data()?['balance'] as num?)?.toDouble() ?? 0;
        if (balance < 50.0) {
          await blockDriver(driverId);
          return true; // was blocked
        }
      }
    } catch (_) {
      // ignore
    }
    return false;
  }

  Future<void> updateDriverProfile(String driverId, String name, String phone) async {
    final user = FirebaseAuth.instance.currentUser;
    if (user != null && user.uid == driverId) {
      final newEmail = '$phone@tawseel.app';
      if (user.email != newEmail) {
        try {
          // Update the email in Firebase Auth so the driver can login with the new phone number
          // ignore: deprecated_member_use
          await user.updateEmail(newEmail);
        } catch (e) {
          // Log or silently ignore so that we at least update the Firestore profile.
          // Re-auth would be needed to actually change the login email, but we allow 
          // changing the contact phone number in the app database.
        }
      }
    }

    await _firestore.collection('drivers').doc(driverId).update({
      'name': name,
      'phone': phone,
    });
    
    // Also strictly update the general users collection for consistency
    await _firestore.collection('users').doc(driverId).update({
      'name': name,
      'phone': phone,
    });
  }

  Future<void> migrateDriverDocument(String driverId) async {
    final ref = _firestore.collection('drivers').doc(driverId);
    final snap = await ref.get();
    if (!snap.exists || snap.data() == null) return;

    final data = snap.data()!;
    final model = DriverModel.fromJson(data, snap.id);

    final payload = model.toJson()
      ..addAll({
        // Ensure defaults are present
        'totalTrips': (data['totalTrips'] as num?)?.toInt() ?? model.totalTrips,
        'walletBalance': (data['walletBalance'] as num?)?.toDouble() ?? model.walletBalance,
        'commissionRate': (data['commissionRate'] as num?)?.toDouble() ?? model.commissionRate,
      });

    await ref.set(payload, SetOptions(merge: true));
  }

  Future<void> migrateAllDrivers() async {
    final snapshot = await _firestore.collection('drivers').get();
    for (final doc in snapshot.docs) {
      await migrateDriverDocument(doc.id);
    }
  }
}

و

import 'dart:async';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:flutter/foundation.dart';

class PushNotificationService {
  final FirebaseMessaging _fcm = FirebaseMessaging.instance;
  final FirebaseFirestore _firestore = FirebaseFirestore.instance;
  
  static final FlutterLocalNotificationsPlugin _localNotifications = 
      FlutterLocalNotificationsPlugin();

  /// Initialize notifications
  Future<void> initNotifications() async {
    // 1. Request permission
    NotificationSettings settings = await _fcm.requestPermission(
      alert: true,
      badge: true,
      sound: true,
    );

    if (settings.authorizationStatus == AuthorizationStatus.authorized) {
      if (kDebugMode) {
        print('User granted permission');
      }
    }

    // 2. Initialize local notifications (for foreground messages)
    const AndroidInitializationSettings androidSettings =
        AndroidInitializationSettings('@mipmap/ic_launcher');
    
    const InitializationSettings initSettings = InitializationSettings(
      android: androidSettings,
    );

    await _localNotifications.initialize(
      settings: initSettings,
      onDidReceiveNotificationResponse: (NotificationResponse details) {
        // Handle notification tap
      },
    );

    // 3. Listen for foreground messages
    FirebaseMessaging.onMessage.listen((RemoteMessage message) {
      _showLocalNotification(message);
    });

    // 4. Handle notification tap when app is in background/terminated
    FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage message) {
      if (kDebugMode) {
        print('Notification tapped: ${message.notification?.title}');
      }
    });
  }

  /// Get and save the FCM token for the user
  Future<void> updateToken(String userId, String role) async {
    String? token = await _fcm.getToken();
    if (token != null) {
      final collection = role == 'driver' ? 'drivers' : 'users';
      await _firestore.collection(collection).doc(userId).set({
        'fcmToken': token,
        'lastTokenUpdate': FieldValue.serverTimestamp(),
      }, SetOptions(merge: true));
    }
  }

  /// Show a local notification when the app is in the foreground
  Future<void> _showLocalNotification(RemoteMessage message) async {
    RemoteNotification? notification = message.notification;
    AndroidNotification? android = message.notification?.android;

    if (notification != null && android != null) {
      await showLocalNotification(
        title: notification.title ?? '',
        body: notification.body ?? '',
      );
    }
  }

  /// Show a custom local notification from the app
  static Future<void> showLocalNotification({
    required String title,
    required String body,
  }) async {
    await _localNotifications.show(
      id: DateTime.now().millisecond,
      title: title,
      body: body,
      notificationDetails: const NotificationDetails(
        android: AndroidNotificationDetails(
          'high_importance_channel',
          'High Importance Notifications',
          importance: Importance.max,
          priority: Priority.high,
          icon: '@mipmap/ic_launcher',
        ),
      ),
    );
  }
}

/// Background message handler
@pragma('vm:entry-point')
Future<void> firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  // If you're going to use other Firebase services in the background, such as Firestore,
  // make sure you call `Firebase.initializeApp()` before using them.
  if (kDebugMode) {
    print('Handling a background message: ${message.messageId}');
  }
}


و


import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/trip.dart';
import 'location_service.dart';

class TripService {
  final FirebaseFirestore _firestore;

  TripService({FirebaseFirestore? firestore})
      : _firestore = firestore ?? FirebaseFirestore.instance;

  /// Stream nearby pending requests filtered by driver's GPS proximity.
  /// [driverLat] and [driverLng] must be provided for proximity filtering.
  /// If no driver location is available, falls back to showing all pending.
  Stream<List<TripModel>> streamNearbyRequests({
    double? driverLat,
    double? driverLng,
    double radiusKm = LocationService.maxRadiusKm,
  }) {
    return _firestore
        .collection('trips')
        .where('status', isEqualTo: 'searching')
        .snapshots()
        .asyncMap((snapshot) async {
      final allRequests = <TripModel>[];
      for (final doc in snapshot.docs) {
        final baseTrip = TripModel.fromJson(doc.data(), doc.id);
        String clientName = baseTrip.clientName;
        String clientPhone = baseTrip.clientPhone;
        if (baseTrip.userId.isNotEmpty) {
          final userDoc =
              await _firestore.collection('users').doc(baseTrip.userId).get();
          if (userDoc.exists) {
            final data = userDoc.data() ?? <String, dynamic>{};
            clientName = (data['name'] ?? clientName).toString();
            clientPhone = (data['phone'] ?? clientPhone).toString();
          }
        }
        allRequests.add(
          baseTrip.copyWith(
            clientName: clientName,
            clientPhone: clientPhone,
          ),
        );
      }

      // If driver location is unknown, show all pending requests
      if (driverLat == null || driverLng == null) return allRequests;

      // Filter: only requests whose pickup is within radiusKm
      return allRequests.where((trip) {
        // If the trip has no GPS coords → show it to everyone (fallback)
        if (!trip.hasPickupCoords) return true;

        return LocationService.isDriverNearPickup(
          driverLat: driverLat,
          driverLng: driverLng,
          pickupLat: trip.pickupLat!,
          pickupLng: trip.pickupLng!,
          radiusKm: radiusKm,
        );
      }).toList();
    });
  }

  /// Atomically accept a trip in the same `trips` collection.
  Future<void> acceptTrip(String tripId, String driverId, {String? clientName, String? clientPhone}) async {
    final tripRef = _firestore.collection('trips').doc(tripId);
    final walletRef = _firestore.collection('wallets').doc(driverId);
    final driverRef = _firestore.collection('drivers').doc(driverId);

    await _firestore.runTransaction((transaction) async {
      final tripSnapshot = await transaction.get(tripRef);
      if (!tripSnapshot.exists) throw Exception('Trip not found or already taken');

      final tripData = tripSnapshot.data()!;
      final currentStatus = (tripData['status'] ?? '').toString();
      final existingDriverId =
          (tripData['driverId'] ?? tripData['driver_id'] ?? '').toString();
      if (currentStatus != 'searching' || existingDriverId.isNotEmpty) {
        throw Exception('Trip already taken');
      }

      final walletSnapshot = await transaction.get(walletRef);
      final driverSnapshot = await transaction.get(driverRef);

      final estimatedPrice = (tripData['price'] as num?)?.toDouble() ?? 0.0;
      final walletBalance = walletSnapshot.exists ? ((walletSnapshot.data()?['balance'] as num?)?.toDouble() ?? 0.0) : 0.0;
      final commissionRate = driverSnapshot.exists ? ((driverSnapshot.data()?['commissionRate'] as num?)?.toDouble() ?? 0.15) : 0.15;

      final requiredBalance = estimatedPrice * commissionRate;
      if (walletBalance < requiredBalance) {
        throw Exception(
            'Solde insuffisant. Vous avez besoin d\'au moins ${requiredBalance.toStringAsFixed(2)} MRU pour accepter cette course (${(commissionRate * 100).toInt()}%).');
      }

      final updateData = <String, dynamic>{
        'status': 'accepted',
        'driverId': driverId,
        'driver_id': driverId,
        'accepted_at': FieldValue.serverTimestamp(),
      };
      
      if (clientName != null && clientName.isNotEmpty) {
        updateData['clientName'] = clientName;
      }
      if (clientPhone != null && clientPhone.isNotEmpty) {
        updateData['clientPhone'] = clientPhone;
      }

      transaction.update(tripRef, updateData);
    });
  }

  /// Update trip status in 'trips' collection
  Future<void> updateTripStatus(String tripId, String status) async {
    await _firestore.collection('trips').doc(tripId).update({
      'status': status,
    });
  }

  /// Complete a trip with fare and commission calculations.
  /// Atomically deducts the commission from the driver's wallet.
  Future<void> completeTrip(String tripId, double fare, double commission) async {
    final tripRef = _firestore.collection('trips').doc(tripId);

    await _firestore.runTransaction((transaction) async {
      final tripSnapshot = await transaction.get(tripRef);
      if (!tripSnapshot.exists) throw Exception('Trip not found');

      final driverId = tripSnapshot.data()?['driverId'] as String?;
      if (driverId == null) throw Exception('No driver assigned to this trip');

      final walletRef = _firestore.collection('wallets').doc(driverId);
      final driverRef = _firestore.collection('drivers').doc(driverId);

      // Update trip details
      transaction.update(tripRef, {
        'status': 'completed',
        'fare': fare,
        'commission': commission,
        'completedAt': DateTime.now().toIso8601String(),
      });

      // Deduct commission from driver wallet
      transaction.set(walletRef, {
        'balance': FieldValue.increment(-commission),
      }, SetOptions(merge: true));

      // Increment total trips in driver profile
      transaction.update(driverRef, {
        'totalTrips': FieldValue.increment(1),
      });
    });
  }

  /// Cancel a trip
  Future<void> cancelTrip(String tripId) async {
    await _firestore.collection('trips').doc(tripId).update({
      'status': 'cancelled',
    });
  }

  /// Stream the driver's active trip (accepted or in_progress) from 'trips' collection.
  /// Fetches all active and picks the most recent one by acceptance date.
  Stream<TripModel?> streamActiveTrip(String driverId) {
    return _firestore
        .collection('trips')
        .where('driverId', isEqualTo: driverId)
        .where('status', whereIn: ['accepted', 'arrived', 'in_progress'])
        .snapshots()
        .map((snapshot) {
      if (snapshot.docs.isEmpty) return null;
      
      // Map docs to models
      final trips = snapshot.docs
          .map((doc) => TripModel.fromJson(doc.data(), doc.id))
          .toList();

      // Sort by accepted_at descending (most recent first)
      trips.sort((a, b) {
        final aTime = a.acceptedAt ?? a.createdAt ?? DateTime(2000);
        final bTime = b.acceptedAt ?? b.createdAt ?? DateTime(2000);
        return bTime.compareTo(aTime);
      });

      return trips.first;
    });
  }

  /// Stream completed trips for history from 'trips' collection
  Stream<List<TripModel>> streamTripHistory(String driverId) {
    return _firestore
        .collection('trips')
        .where('driverId', isEqualTo: driverId)
        .where('status', isEqualTo: 'completed')
        .snapshots()
        .map((snapshot) {
      return snapshot.docs
          .map((doc) => TripModel.fromJson(doc.data(), doc.id))
          .toList();
    });
  }
}

و

class AppLocalizations {
  static const Map<String, Map<String, String>> _localizedValues = {
    'fr': {
      'personal_info': 'Informations personnelles',
      'notifications': 'Notifications',
      'security': 'Sécurité',
      'language': 'Langue',
      'help_support': 'Aide & Support',
      'about': 'À propos',
      'logout': 'Déconnexion',
      'availability': 'Disponibilité',
      'online': 'En ligne',
      'offline': 'Hors ligne',
      'french': 'Français',
      'arabic': 'العربية',
      'choose_language': 'Choisir la langue',
      'settings': 'Réglages',
      'requests_list': 'Liste des commandes',
      'history': 'Historique',
      'you_are_offline': 'Vous êtes HORS LIGNE.',
      'activate_to_receive': 'Activez pour recevoir des commandes',
      'access_suspended': 'Accès suspendu',
      'waiting_for_requests': 'En attente de commandes...',
      'error': 'Erreur: ',
      'retry': 'Réessayer',
      'account_blocked_msg': 'Votre compte est bloqué. Veuillez recharger votre solde pour continuer.',
      'low_balance_msg': 'Solde bas. Rechargez bientôt.',
      'my_wallet': 'Mon Portefeuille',
      'current_balance': 'Solde actuel',
      'insufficient_balance_recharge_msg': 'Solde insuffisant. Rechargez pour recevoir des courses.',
      'withdraw': 'Retirer',
      'total_earned': 'Total gagné',
      'commission_label': 'Commission',
      'rating_label': 'Note',
      'recent_transactions': 'Transactions récentes',
      'no_transactions': 'Aucune transaction',
      'gross_income': 'Revenu brut',
      'net_income': 'Revenu net',
      'no_completed_trips': 'Aucune course terminée',
      'trip_completed': 'Course terminée!',
      'back': 'Retour',
      'navigation': 'Navigation',
      'pickup_point': 'Départ',
      'destination_point': 'Arrivée',
      'call_client': 'Appeler le client',
      'cancel': 'Annuler',
      'arrived_at_point': 'ARRIVÉ AU POINT',
      'start_trip': 'COMMENCER LA COURSE',
      'finish_trip': 'TERMINER LA COURSE',
      'waiting': 'EN ATTENTE...',
      'finish_trip_q': 'Terminer la course?',
      'confirm_finish_trip_msg': 'Confirmez la fin de la course. Le montant sera ajouté à votre portefeuille.',
      'confirm': 'Confirmer',
      'cancel_trip_q': 'Annuler la course?',
      'confirm_cancel_trip_msg': 'Êtes-vous sûr de vouloir annuler cette course?',
      'no': 'Non',
      'yes_cancel': 'Oui, annuler',
      // Security
      'change_password': 'Changer le mot de passe',
      'current_password': 'Mot de passe actuel',
      'new_password': 'Nouveau mot de passe',
      'confirm_password': 'Confirmer le mot de passe',
      'password_changed': 'Mot de passe modifié avec succès',
      'wrong_current_password': 'Mot de passe actuel incorrect',
      'relogin_required': 'Veuillez vous reconnecter pour effectuer cette action',
      'passwords_not_match': 'Les mots de passe ne correspondent pas',
      'password_too_short': 'Le mot de passe doit contenir au moins 6 caractères',
      'error_occurred': 'Une erreur s\'est produite',
      'security_tips': 'Conseils de sécurité',
      'tip_strong_password': 'Utilisez un mot de passe fort avec des chiffres et symboles',
      'tip_dont_share': 'Ne partagez jamais votre mot de passe avec personne',
      'tip_logout_public': 'Déconnectez-vous toujours sur les appareils publics',
      // Help & Support
      'contact_us': 'Contactez-nous',
      'call_us': 'Appeler',
      'email_us': 'Email',
      'faq': 'Questions fréquentes',
      'report_problem': 'Signaler un problème',
      'report_problem_desc': 'Vous rencontrez un problème ? Envoyez-nous un rapport et nous vous répondrons dans les 24h.',
      'send_report': 'Envoyer un rapport',
      'faq_q1': 'Comment activer mon compte chauffeur ?',
      'faq_a1': 'Après votre inscription, rechargez votre portefeuille avec au moins 50 MRU pour activer votre compte et commencer à recevoir des commandes.',
      'faq_q2': 'Pourquoi je ne reçois pas de commandes ?',
      'faq_a2': 'Vérifiez que vous êtes en ligne et que votre solde est supérieur à 50 MRU. Si le problème persiste, redémarrez l\'application.',
      'faq_q3': 'Comment est calculée la commission ?',
      'faq_a3': 'La commission est calculée sur la base du tarif de la course. Le taux varie selon votre contrat. Consultez votre profil pour voir votre taux actuel.',
      'faq_q4': 'Comment recharger mon portefeuille ?',
      'faq_a4': 'Contactez notre support ou un agent Tawseel dans votre région pour effectuer un rechargement.',
      // About
      'about_app': 'À propos de Tawseel',
      'about_app_desc': 'Tawseel est une application de livraison et transport qui connecte les chauffeurs avec les clients en Mauritanie. Notre plateforme offre une solution simple, rapide et sécurisée pour vos besoins de transport.',
      'our_mission': 'Notre mission',
      'our_mission_desc': 'Offrir un service de transport fiable, abordable et accessible à tous, tout en garantissant un revenu stable et équitable pour nos chauffeurs partenaires.',
      'drivers': 'Chauffeurs',
      'trips_done': 'Courses',
      'legal': 'Mentions légales',
      'terms_of_service': 'Conditions d\'utilisation',
      'privacy_policy': 'Politique de confidentialité',
      'all_rights_reserved': 'Tous droits réservés.',
      'made_with_love': 'Fait avec ❤️ en Mauritanie',
      'app_tagline': 'Transport rapide et fiable en Mauritanie',
      // Common
      'save': 'Enregistrer',
      'fill_all_fields': 'Veuillez remplir tous les champs',
      'wallet': 'Portefeuille',
      'trips': 'courses',
      'full_name': 'Nom complet',
      'phone': 'Numéro de téléphone',
      'profile_updated': 'Profil mis à jour',
    },
    'ar': {
      'personal_info': 'المعلومات الشخصية',
      'notifications': 'الإشعارات',
      'security': 'الأمان',
      'language': 'اللغة',
      'help_support': 'المساعدة والدعم',
      'about': 'حول التطبيق',
      'logout': 'تسجيل الخروج',
      'availability': 'حالة التوافر',
      'online': 'متصل',
      'offline': 'غير متصل',
      'french': 'الفرنسية',
      'arabic': 'العربية',
      'choose_language': 'اختر اللغة',
      'settings': 'الإعدادات',
      'requests_list': 'قائمة الطلبات',
      'history': 'السجل',
      'you_are_offline': 'أنت غير متصل.',
      'activate_to_receive': 'قم بالتفعيل لاستقبال الطلبات',
      'access_suspended': 'تم تعليق الوصول',
      'waiting_for_requests': 'في انتظار الطلبات...',
      'error': 'خطأ: ',
      'retry': 'إعادة المحاولة',
      'account_blocked_msg': 'حسابك معلق. يرجى شحن رصيدك للمتابعة.',
      'low_balance_msg': 'رصيد منخفض. يرجى الشحن قريباً.',
      'my_wallet': 'محفظتي',
      'current_balance': 'الرصيد الحالي',
      'insufficient_balance_recharge_msg': 'رصيد غير كافٍ. قم بالشحن لاستقبال الطلبات.',
      'withdraw': 'سحب',
      'total_earned': 'إجمالي الأرباح',
      'commission_label': 'العمولة',
      'rating_label': 'التقييم',
      'recent_transactions': 'المعاملات الأخيرة',
      'no_transactions': 'لا توجد معاملات',
      'gross_income': 'الدخل الإجمالي',
      'net_income': 'الدخل الصافي',
      'no_completed_trips': 'لا توجد رحلات مكتملة',
      'trip_completed': 'انتهت الرحلة!',
      'back': 'رجوع',
      'navigation': 'الملاحة',
      'pickup_point': 'الإنطلاق',
      'destination_point': 'الوصول',
      'call_client': 'الاتصال بالزبون',
      'cancel': 'إلغاء',
      'arrived_at_point': 'وصلت إلى نقطة الإنطلاق',
      'start_trip': 'بدء الرحلة',
      'finish_trip': 'إنهاء الرحلة',
      'waiting': 'قيد الانتظار...',
      'finish_trip_q': 'إنهاء الرحلة؟',
      'confirm_finish_trip_msg': 'تأكيد إنهاء الرحلة. سيتم إضافة المبلغ إلى محفظتك.',
      'confirm': 'تأكيد',
      'cancel_trip_q': 'إلغاء الرحلة؟',
      'confirm_cancel_trip_msg': 'هل أنت متأكد أنك تريد إلغاء هذه الرحلة؟',
      'no': 'لا',
      'yes_cancel': 'نعم، إلغاء',
      // Security
      'change_password': 'تغيير كلمة المرور',
      'current_password': 'كلمة المرور الحالية',
      'new_password': 'كلمة المرور الجديدة',
      'confirm_password': 'تأكيد كلمة المرور',
      'password_changed': 'تم تغيير كلمة المرور بنجاح',
      'wrong_current_password': 'كلمة المرور الحالية غير صحيحة',
      'relogin_required': 'يرجى تسجيل الدخول مرة أخرى لإجراء هذا الإجراء',
      'passwords_not_match': 'كلمتا المرور غير متطابقتين',
      'password_too_short': 'يجب أن تحتوي كلمة المرور على 6 أحرف على الأقل',
      'error_occurred': 'حدث خطأ',
      'security_tips': 'نصائح الأمان',
      'tip_strong_password': 'استخدم كلمة مرور قوية تحتوي على أرقام ورموز',
      'tip_dont_share': 'لا تشارك كلمة مرورك مع أحد أبداً',
      'tip_logout_public': 'قم دائماً بتسجيل الخروج على الأجهزة العامة',
      // Help & Support
      'contact_us': 'اتصل بنا',
      'call_us': 'اتصل',
      'email_us': 'البريد الإلكتروني',
      'faq': 'الأسئلة الشائعة',
      'report_problem': 'الإبلاغ عن مشكلة',
      'report_problem_desc': 'هل تواجه مشكلة؟ أرسل لنا تقريراً وسنرد عليك خلال 24 ساعة.',
      'send_report': 'إرسال تقرير',
      'faq_q1': 'كيف أفعّل حسابي كسائق؟',
      'faq_a1': 'بعد التسجيل، اشحن محفظتك بـ 50 أوقية على الأقل لتفعيل حسابك وبدء استقبال الطلبات.',
      'faq_q2': 'لماذا لا أتلقى طلبات؟',
      'faq_a2': 'تأكد من أنك متصل وأن رصيدك يزيد عن 50 أوقية. إذا استمرت المشكلة، أعد تشغيل التطبيق.',
      'faq_q3': 'كيف تُحسب العمولة؟',
      'faq_a3': 'تُحسب العمولة على أساس أجر الرحلة. يختلف المعدل حسب عقدك. راجع ملفك الشخصي لمعرفة معدلك الحالي.',
      'faq_q4': 'كيف أشحن محفظتي؟',
      'faq_a4': 'تواصل مع الدعم أو وكيل تواصيل في منطقتك لإجراء عملية الشحن.',
      // About
      'about_app': 'حول تواصيل',
      'about_app_desc': 'تواصيل هو تطبيق للتوصيل والنقل يربط السائقين بالعملاء في موريتانيا. توفر منصتنا حلاً بسيطاً وسريعاً وآمناً لاحتياجات النقل الخاصة بك.',
      'our_mission': 'مهمتنا',
      'our_mission_desc': 'تقديم خدمة نقل موثوقة وبأسعار معقولة ومتاحة للجميع، مع ضمان دخل مستقر وعادل لسائقينا الشركاء.',
      'drivers': 'سائق',
      'trips_done': 'رحلة',
      'legal': 'المعلومات القانونية',
      'terms_of_service': 'شروط الاستخدام',
      'privacy_policy': 'سياسة الخصوصية',
      'all_rights_reserved': 'جميع الحقوق محفوظة.',
      'made_with_love': 'صُنع بـ ❤️ في موريتانيا',
      'app_tagline': 'نقل سريع وموثوق في موريتانيا',
      // Common
      'save': 'حفظ',
      'fill_all_fields': 'يرجى ملء جميع الحقول',
      'wallet': 'المحفظة',
      'trips': 'رحلات',
      'full_name': 'الاسم الكامل',
      'phone': 'رقم الهاتف',
      'profile_updated': 'تم تحديث الملف الشخصي',
    },
  };

  static String translate(String key, String locale) {
    return _localizedValues[locale]?[key] ?? key;
  }
}


و

import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:flutter/material.dart';

class LocalizationCubit extends Cubit<Locale> {
  static const String _langKey = 'app_lang';
  final SharedPreferences prefs;

  LocalizationCubit(this.prefs) : super(const Locale('fr')) {
    _loadLang();
  }

  void _loadLang() {
    final lang = prefs.getString(_langKey);
    if (lang != null) {
      emit(Locale(lang));
    }
  }

  Future<void> changeLanguage(String langCode) async {
    await prefs.setString(_langKey, langCode);
    emit(Locale(langCode));
  }
}

و

import 'package:flutter/material.dart';

class AppColors {
  static const Color gold = Color(0xFFF5D142);

  static Color scaffold(BuildContext context) =>
      Theme.of(context).scaffoldBackgroundColor;

  static Color surface(BuildContext context) =>
      Theme.of(context).colorScheme.surface;

  static Color onSurface(BuildContext context) =>
      Theme.of(context).colorScheme.onSurface;

  static Color border(BuildContext context) =>
      Theme.of(context).brightness == Brightness.dark
          ? Colors.white10
          : Colors.black12;

  static Color subtext(BuildContext context) =>
      Theme.of(context).brightness == Brightness.dark
          ? Colors.grey.shade500
          : Colors.grey.shade600;

  static bool isDark(BuildContext context) =>
      Theme.of(context).brightness == Brightness.dark;
}

